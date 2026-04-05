# Loan Validator Agent

## Purpose

The Loan Validator is a LlamaIndex sub-agent responsible for **Phase 1 (Loan Tape Acquisition)** and the statistical screening component of **Phase 2 (Due Diligence)**. It processes a `LoanTape` and produces a validated, MISMO-normalized dataset ready for Knowledge Graph ingestion, along with a defect report.

---

## Activation

Invoke this agent when:
- A new loan tape file has been received and needs ingestion
- A user asks to "validate a tape", "run QC on a tape", or "check tape quality"
- The ingestion pipeline triggers tape validation

Do **not** invoke this agent for post-securitization data (remittance reports, servicer data) — use the monitoring pipeline instead.

---

## Inputs

```python
class LoanTapeValidationRequest(BaseModel):
    tape_id: str                   # Assigned before invoking agent
    file_path: str                 # Path to the raw tape file (CSV, pipe-delimited, Excel)
    originator_nmls_id: str        # Used to look up originator history in KG
    received_date: date            # Date tape was received from seller
    expected_loan_count: int | None  # If known from deal term sheet
```

---

## Validation Steps (in order)

### Step 1 — File Parsing & Field Detection
- Detect delimiter (CSV, pipe, tab, Excel)
- Identify column headers; map to MISMO standard field names using the field alias table in `neo4j-conventions.md`
- Flag any columns that cannot be mapped to a known MISMO field — surface as `UNKNOWN_FIELD` warnings
- Report total row count vs. `expected_loan_count` (if provided)

### Step 2 — Required Field Presence Check
The following fields are **required**. A tape missing any of these fails with `CRITICAL` defect:

| MISMO Field | Required | Notes |
|-------------|----------|-------|
| `loanId` | Yes | Must be unique within the tape |
| `upb` | Yes | — |
| `fico` | Yes | — |
| `ltv` | Yes | — |
| `dti` | Yes | — |
| `originationDate` | Yes | — |
| `propertyState` | Yes | — |
| `occupancyType` | Yes | `PRIMARY`, `SECOND_HOME`, `INVESTMENT` |
| `loanPurpose` | Yes | `PURCHASE`, `RATE_TERM_REFI`, `CASH_OUT_REFI` |
| `documentationType` | Yes | `FULL_DOC`, `ALT_A`, `DSCR` |
| `brokerId` or `originatorId` | Yes | At least one must be present |

### Step 3 — Field-Level Data Quality Checks
Apply invariants from `graph-schema-rules.md § Data Quality Invariants`:

| Field | Valid Range | On Failure |
|-------|-------------|------------|
| `fico` | [300, 850] | Flag as `DEFECT_FLAGGED` |
| `ltv` | (0.0, 2.0] | Flag as `DEFECT_FLAGGED` |
| `cltv` | (0.0, 3.0], must be ≥ `ltv` | Flag as `DEFECT_FLAGGED` |
| `dti` | (0.0, 1.5] | Flag; dti > 0.55 triggers `HIGH_DTI` review flag |
| `upb` | > 0 | Flag as `DEFECT_FLAGGED` |
| `originationDate` | Not future, not before 1970-01-01 | Flag as `DEFECT_FLAGGED` |

### Step 4 — Logical Consistency Checks
- `cltv >= ltv` for every loan (if both fields present)
- `originationDate <= receivedDate`
- If `loanPurpose == CASH_OUT_REFI`, a `cashOutAmount` field should be present
- If `documentationType == DSCR`, `dti` should be null or 0 (DSCR loans have no personal income verification)
- `occupancyType == INVESTMENT` loans with `fico > 780` and `ltv > 0.90` trigger `OCCUPANCY_MISMATCH` review flag

### Step 5 — Pool-Level Statistical Validation
Compute and report pool statistics. Flag pool-level anomalies:

| Metric | Anomaly Threshold | Signal Type |
|--------|------------------|-------------|
| % loans with DTI > 0.55 | ≥ 10% of pool | `HIGH_DTI_CLUSTER` |
| Weighted average LTV | ≥ 0.95 | `HIGH_WALTV_WARNING` |
| Single-state UPB concentration | > 40% | `GEOGRAPHIC_CONCENTRATION` |
| Single-MSA UPB concentration | > 20% | `GEOGRAPHIC_CONCENTRATION` |
| EPD rate on originator's prior pools (from KG) | ≥ 3% | `ORIGINATOR_HIGH_EPD_HISTORY` |

### Step 6 — Originator Intelligence Lookup
Query the Knowledge Graph for the originator's historical performance:

```cypher
MATCH (o:Originator {nmlsId: $nmlsId})
OPTIONAL MATCH (o)<-[:ORIGINATED_BY]-(l:Loan)
RETURN
  o.historicalDefectRate AS defectRate,
  o.avgDelinquencyRate   AS avgDelinqRate,
  count(l)               AS priorLoanCount
```

Surface the following in the validation report:
- Number of prior tapes from this originator in the KG
- Historical defect rate (if available)
- Any open fraud cases involving this originator

---

## Output

```python
class LoanTapeValidationResult(BaseModel):
    tape_id: str
    validation_status: Literal["PASSED", "PASSED_WITH_FLAGS", "FAILED"]
    total_loans: int
    defect_count: int            # Loans with at least one DEFECT_FLAGGED issue
    review_flag_count: int       # Loans with review flags (not defects)
    pool_statistics: PoolStats
    originator_history: OriginatorHistory | None
    defect_report: list[LoanDefect]   # One entry per defective loan
    pool_anomalies: list[PoolAnomaly] # Pool-level signals
    recommended_action: Literal[
        "PROCEED_TO_GRAPH_INGESTION",
        "PROCEED_WITH_ENHANCED_REVIEW",
        "ESCALATE_TO_DUE_DILIGENCE_TEAM",
        "REJECT_TAPE"
    ]
```

### `recommended_action` Decision Logic

| Condition | Action |
|-----------|--------|
| No defects, no anomalies | `PROCEED_TO_GRAPH_INGESTION` |
| < 5% defect rate, no CRITICAL anomalies | `PROCEED_WITH_ENHANCED_REVIEW` |
| 5–15% defect rate OR any `HIGH_DTI_CLUSTER` / `GEOGRAPHIC_CONCENTRATION` | `ESCALATE_TO_DUE_DILIGENCE_TEAM` |
| > 15% defect rate OR missing required fields on > 10% of loans | `REJECT_TAPE` |
| Originator has open fraud case in KG | `ESCALATE_TO_DUE_DILIGENCE_TEAM` (always) |

---

## Knowledge Graph Writes

After validation, the agent writes to the KG:

1. Create or update `(:LoanTape)` node with validation results
2. Create `(:Loan)` nodes for all loans (including defect-flagged ones — they must be in the graph for audit)
3. Create or `MERGE` `(:Originator)` node
4. Create `[:ORIGINATED_BY]` relationships
5. Create `[:BROKERED_BY]` relationships (if broker IDs present)
6. Create `(:FraudSignal)` nodes for any pool-level anomalies that constitute fraud signals

All writes use parameterized Cypher. Follow all conventions in `neo4j-conventions.md`.

---

## HITL Gate

**This agent does NOT make any fraud determinations or broker blacklisting decisions.**

If pool-level anomalies reach fraud signal thresholds, the agent:
1. Creates the `FraudSignal` node in the KG
2. Returns `recommended_action = ESCALATE_TO_DUE_DILIGENCE_TEAM`
3. Stops. A human investigator (or the Fraud Signal Detector agent, explicitly invoked) takes over.

The agent **never** creates a `FraudCase` or modifies a `BrokerRiskProfile` directly.
