---
description: >
  Schema governance rules, cross-context constraints, and immutability invariants
  for the MBS Intelligence Platform Knowledge Graph.
applyTo: "**/*.cypher, **/*.py, infrastructure/schema/**"
---

# Graph Schema Rules — MBS Intelligence Platform

These rules govern how the Knowledge Graph schema evolves and how data integrity is
maintained across bounded context boundaries. They complement `neo4j-conventions.md`
(which covers syntax) — this file covers **governance and invariants**.

---

## The Golden Rule

> **No new Neo4j node label or relationship type may appear in code before it is
> declared in `memory/mbs-domain-model.md`.**

The domain model is the contract. Code implements the contract. If you need a new
schema element, update the domain model first (with ADR if it affects shared kernel
or multiple contexts), then write the code.

---

## Schema Change Process

### 1. Assess Scope

| Change | Process Required |
|--------|-----------------|
| New property on existing node/rel | Update domain model (patch bump), then code |
| New node label owned by one context | Update domain model (minor bump) + ADR, then code |
| New relationship type within one context | Update domain model (minor bump), then code |
| New node label in Shared Kernel | Update domain model (minor bump) + ADR + cross-context review |
| New relationship crossing context boundaries | ADR required, domain model update, then code |
| Removing or renaming any label/rel type | ADR required — treat as a breaking change |

### 2. ADR Location
Architecture Decision Records live in `architecture/adr/`.
File naming: `ADR-NNN-short-description.md` (e.g., `ADR-004-add-servicer-node.md`).

### 3. Index Declaration
Every new node label requires corresponding index declarations in
`infrastructure/schema/indexes.cypher` before first use in production.

---

## Cross-Context Relationship Rules

Bounded context A may **read** nodes owned by bounded context B.
Bounded context A may **not write properties** to nodes owned by bounded context B.

### Ownership Map

| Node Label | Owner | Who May Write |
|------------|-------|---------------|
| `(:Loan)` | Loan Tape Ingestion | `loan_tape_pipeline` only |
| `(:Originator)` | Loan Tape Ingestion | `loan_tape_pipeline` only |
| `(:Broker)` | Fraud Intelligence | `fraud_intelligence_network` only |
| `(:Appraiser)` | Fraud Intelligence | `fraud_intelligence_network` only |
| `(:FraudCase)` | Fraud Intelligence | `fraud_intelligence_network` only (HITL-gated) |
| `(:FraudSignal)` | Fraud Intelligence | `fraud_intelligence_network` only |
| `(:Pool)` | Post-Securitization Monitoring | `early_warning_engine` only |
| `(:EarlyWarningAlert)` | Post-Securitization Monitoring | `early_warning_engine` only |
| `(:MacroCycleSnapshot)` | Post-Securitization Monitoring | `early_warning_engine` only |
| `(:Property)` | Shared Kernel | Any context may write — coordinate changes |
| `(:MSAMetrics)` | Shared Kernel | `early_warning_engine` (macro cycle only) |

### Cross-Context Read Pattern
When a service needs data from another context's nodes, it reads via the KG service
(`neo4j_kg_service`) — never directly imports another context's Pydantic models.

---

## Shared Kernel Contracts

The Shared Kernel contains schema elements consumed by two or more bounded contexts.
Changes require cross-context coordination before implementation.

| Element | Consuming Contexts | Stability |
|---------|--------------------|-----------|
| `(:Property)` node + `propertyId` | Loan Tape Ingestion, Fraud Intelligence, Post-Securitization Monitoring | HIGH — do not change lightly |
| `[:SECURES]` relationship | Loan Tape Ingestion, Fraud Intelligence | HIGH |
| `(:MSAMetrics)` node | Post-Securitization Monitoring, Fraud Intelligence | MEDIUM |
| `[:LOCATED_IN_MSA]` relationship | Post-Securitization Monitoring, Fraud Intelligence | MEDIUM |
| `propertyId` hashing scheme | All contexts | HIGH — changing breaks cross-context joins |

### propertyId Hashing Contract
`propertyId` is a deterministic hash derived from the property's address components.
The hash function is defined once in `shared/property_id.py` and **must not be changed**
without a migration plan. All contexts depend on this ID being stable.

```python
# shared/property_id.py — the one canonical implementation
def compute_property_id(street: str, city: str, state: str, zip_code: str) -> str:
    # SHA-256 of normalized (uppercase, stripped) address components
    ...
```

---

## Immutability Invariants

### Fraud Nodes — Append Only
The following node types are **immutable once created**:

- `(:FraudSignal)` — never SET properties after creation
- `(:FraudCase)` — status changes recorded as `(:FraudCaseEvent)` nodes, not property mutations
- `(:FraudCaseEvent)` — never modified after creation
- Any relationship attached to the above

**Rationale:** Fraud evidence must form a tamper-evident audit chain. Mutation would destroy the integrity of any regulatory or legal review.

**Enforcement:** The `fraud_intelligence_network` service enforces this at the application layer. Neo4j RBAC (when enterprise is available) should restrict WRITE on these labels to the fraud service user only.

### Audit Trail Relationships
All audit trail relationships carry:
- `createdAt: datetime()` — set at write time, never updated
- `createdBy: String` — the agent ID or user ID responsible for the write

### Loan Tape Nodes — Immutable After Validation
`(:LoanTape)` nodes are immutable after `status = 'VALIDATED'`. Property corrections
before validation are permitted; after validation, create a new tape version node.

---

## Data Quality Invariants

These invariants are checked by the loan validator agent before any `(:Loan)` node is written:

| Property | Constraint |
|----------|-----------|
| `fico` | Integer in [300, 850] |
| `ltv` | Float in (0.0, 2.0] — values > 1.0 are high-risk but not impossible |
| `cltv` | Float in (0.0, 3.0] — must be ≥ `ltv` |
| `dti` | Float in (0.0, 1.5] — values > 0.55 trigger review flag |
| `upb` | Float > 0 |
| `originationDate` | ISO date string, not in the future, not before 1970 |
| `nmlsId` (Broker/Originator) | Non-null, non-empty String — required before node creation |

Loans that fail invariant checks are written as `(:Loan {status: 'DEFECT_FLAGGED'})` with
a `defectReason` property — they are **not** dropped from the graph.

---

## Fraud Signal Type Registry

All `FraudSignal.signalType` values must come from this registry.
Do not introduce ad-hoc string values.

| Signal Type | Description | Primary Phase |
|-------------|-------------|---------------|
| `EPD` | Early Payment Default — loan defaulted within first 3 payments | Phase 2 / Phase 8 |
| `AVM_GAP` | Appraised value exceeds AVM estimate by > 15% | Phase 2 / Phase 8 |
| `BROKER_APPRAISER_COOCCURRENCE` | Same broker-appraiser pair on ≥ 3 deals in the graph | Phase 8 |
| `STACKING` | Multiple active mortgages on the same property (same cycle) | Phase 2 / Phase 8 |
| `INCOME_DOCUMENT_MISMATCH` | AI document reader found discrepancy between tape income and W-2/tax returns | Phase 2 |
| `OCCUPANCY_MISMATCH` | Property declared owner-occupied but borrower has multiple active mortgages | Phase 2 / Phase 8 |
| `BROKER_HIGH_RISK_SCORE` | Broker risk score exceeds threshold at time of deal entry | Phase 8 |
| `HIGH_DTI_CLUSTER` | ≥ 10% of pool loans have DTI > 0.55, clustered by originator | Phase 1 / Phase 8 |
| `GEOGRAPHIC_CONCENTRATION` | > 20% of pool UPB in single MSA | Phase 1 / Phase 2 |

---

## Early Warning Alert Type Registry

All `EarlyWarningAlert.alertType` values must come from this registry.

| Alert Type | Trigger Condition | Severity |
|------------|-------------------|----------|
| `DELINQUENCY_RATE_SPIKE` | Pool 30-day DQ rate increased ≥ 150 bps month-over-month | HIGH |
| `ROLL_RATE_ACCELERATION` | 60→90 day roll rate > 20% in current cycle | HIGH |
| `SERVICER_UNDERPERFORMANCE` | Servicer's modification approval rate dropped > 25% vs. 6-month avg | MEDIUM |
| `MACRO_HPA_DECLINE` | MSA HPA dropped > 2% in single month for MSA where pool has > 15% UPB concentration | HIGH |
| `UNEMPLOYMENT_SPIKE` | State unemployment rose > 150 bps in single month for state with > 20% pool UPB | MEDIUM |
| `OC_TEST_APPROACHING_FAIL` | OC ratio within 200 bps of trigger threshold | HIGH |
| `TRIGGER_EVENT_FIRED` | Waterfall trigger event actually breached | CRITICAL |
| `FEMA_DISASTER_DECLARATION` | FEMA declared disaster in geography with > 10% pool UPB exposure | MEDIUM |
| `VINTAGE_UNDERPERFORMANCE` | Pool performing materially worse than same-vintage peers on CDR | MEDIUM |

Alert severity levels: `CRITICAL > HIGH > MEDIUM > LOW`

---

## Graph Projection Policy (GDS)

Named GDS projections follow a lifecycle:
1. **Create** — before algorithm run, scoped to the specific analysis
2. **Run** — execute algorithm
3. **Write results** — persist to node properties immediately
4. **Drop** — always drop the named projection after writing results

Projections must never persist across service restarts or scheduled jobs.
A stale projection is invisible to new data written after its creation.

---

## Prohibited Patterns

These patterns are banned. Flag them in code review:

```cypher
-- Direct property mutation on fraud nodes
MATCH (f:FraudSignal {signalId: $id}) SET f.confidence = $newVal   ← BANNED

-- Cross-context writes (fraud service writing to Loan node properties)
MATCH (l:Loan {loanId: $id}) SET l.fraudScore = $score            ← BANNED (use FraudSignal rel)

-- Unparameterized queries
MATCH (b:Broker {nmlsId: '12345'}) RETURN b                        ← BANNED in application code

-- Deleting audit trail
MATCH (e:FraudCaseEvent) DELETE e                                   ← BANNED

-- Creating new node labels in application code without domain model declaration
CREATE (:RiskBucket {id: $id})                                     ← BANNED (declare first)
```
