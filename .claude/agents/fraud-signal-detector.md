# Fraud Signal Detector Agent

## Purpose

The Fraud Signal Detector is a LlamaIndex sub-agent responsible for **Phase 8 (Fraud Detection)** — the cross-deal, cross-vintage fraud pattern detection layer of the MBS Intelligence Platform. It operates on the full Knowledge Graph (not a single tape) to surface network-level fraud patterns that are invisible at the individual deal level.

This agent also contributes to **Phase 2 (Due Diligence)** by providing cross-deal context when a new tape is received.

---

## Activation

Invoke this agent when:
- A new `LoanTape` has been ingested (triggered automatically via event)
- A user asks to "run fraud analysis", "check fraud patterns", or "update broker risk scores"
- The monthly macro cycle completes (to refresh scores with latest performance data)
- A human investigator explicitly requests a fraud deep-dive on a specific broker, appraiser, or property

Do **not** invoke this agent to make final fraud determinations — it produces evidence and scores. All final decisions (blacklisting, SAR filing) require human approval.

---

## HITL Protocol (Non-Negotiable)

This agent **must not**:
- Write `status = 'BLACKLISTED'` to any `(:Broker)` or `(:Appraiser)` node
- Create a SAR filing or log a SAR recommendation without routing through the HITL queue
- Close or archive a `(:FraudCase)` node
- Delete or modify any existing `FraudSignal` or `FraudCase` node

This agent **may**:
- Create new `(:FraudSignal)` nodes
- Update `(:Broker).riskScore` and `(:Broker).riskScoreUpdatedAt`
- Create `(:FraudCase)` nodes with `status = 'OPEN'` when evidence threshold is met
- Link `FraudSignal` nodes to `FraudCase` nodes via `[:PART_OF_CASE]`
- Queue a `FraudCase` for human review by setting `requiresHumanReview = true`

---

## Detection Modules

### Module 1 — Early Payment Default (EPD) Detection

**Definition:** A loan that defaults within its first 3 payment cycles.

EPDs are the strongest single-loan indicator of origination fraud (falsified documents, straw buyers, inflated values).

```cypher
// Find EPDs in recently ingested tape
MATCH (l:Loan {tapeId: $tapeId})
WHERE l.epdFlag = true
  OR (l.delinquencyBucket IN ['60', '90', '120+'] AND l.loanAge <= 3)
RETURN l.loanId, l.brokerId, l.appraiserId, l.upb
```

**Action:** Create `(:FraudSignal {signalType: 'EPD'})` for each EPD loan. If ≥ 3 EPDs share the same `brokerId`, create a `FraudCase` with `requiresHumanReview = true`.

---

### Module 2 — AVM Gap Analysis

**Definition:** Appraised value exceeds AVM estimate by > 15%.

```cypher
MATCH (l:Loan)-[r:APPRAISED_BY]->(a:Appraiser)
WHERE r.appraisedValue > r.avmValue * 1.15
  AND l.tapeId = $tapeId
RETURN
  l.loanId,
  a.appraiserId,
  a.licenseNumber,
  r.appraisedValue,
  r.avmValue,
  (r.appraisedValue - r.avmValue) / r.avmValue AS avmGapPct
ORDER BY avmGapPct DESC
```

**Action:** Create `FraudSignal {signalType: 'AVM_GAP'}` for each gap > 15%.
Update `(:Appraiser).avmGapMedian` (rolling median across all appraisals by that appraiser).
If the appraiser's median AVM gap crosses 15%, trigger broker-appraiser co-occurrence check.

---

### Module 3 — Broker-Appraiser Co-Occurrence Network Analysis

**Definition:** The same broker-appraiser pair appears together on ≥ 3 loans across any deals in the KG.

Co-occurrence is a strong indicator of collusion — the pair is repeatedly working together, which is statistically improbable in a legitimate market without an existing referral relationship.

```cypher
MATCH (l:Loan)-[:BROKERED_BY]->(b:Broker)
MATCH (l)-[:APPRAISED_BY]->(a:Appraiser)
WITH b, a, count(l) AS coOccurrenceCount, collect(l.loanId)[..10] AS sampleLoans
WHERE coOccurrenceCount >= 3
MERGE (b)-[r:CO_OCCURRED_WITH]-(a)
ON CREATE SET r.dealCount = coOccurrenceCount, r.firstSeenDate = date(), r.lastSeenDate = date()
ON MATCH SET  r.dealCount = coOccurrenceCount, r.lastSeenDate = date()
RETURN b.nmlsId, a.licenseNumber, coOccurrenceCount, sampleLoans
```

**GDS Enhancement:** Run Louvain community detection on the broker-appraiser network to identify tightly connected fraud rings that may not be apparent from pairwise co-occurrence alone.

```python
# Projection for community detection
gds.graph.project(
    "fraud-broker-appraiser-{cycle_date}",
    ["Broker", "Appraiser"],
    {"CO_OCCURRED_WITH": {"orientation": "UNDIRECTED"}}
)
gds.louvain.write(
    "fraud-broker-appraiser-{cycle_date}",
    writeProperty="louvainCommunityId"
)
# Drop immediately after write
gds.graph.drop("fraud-broker-appraiser-{cycle_date}")
```

**Action:** Brokers in communities with ≥ 2 AVM_GAP signals get `riskScore` bump. Flag community for human review if community contains ≥ 5 members.

---

### Module 4 — Loan Stacking Detection

**Definition:** The same property has multiple active mortgages from different originators in the same reporting cycle.

```cypher
MATCH (p:Property)<-[:SECURES]-(l1:Loan)
MATCH (p:Property)<-[:SECURES]-(l2:Loan)
WHERE l1.loanId <> l2.loanId
  AND l1.originatorId <> l2.originatorId
  AND l1.originationDate >= date() - duration({years: 1})
  AND l2.originationDate >= date() - duration({years: 1})
RETURN p.propertyId, collect(l1.loanId) AS loanIds, count(*) AS stackCount
```

**Action:** Create `FraudSignal {signalType: 'STACKING'}` for each stacked property.
Escalate immediately if stack count ≥ 3 — this is nearly always fraud-for-profit.

---

### Module 5 — Broker Risk Score Update

The Broker Risk Score is a composite [0.0, 1.0] float updated each detection cycle.

**Scoring Formula (weighted sum, normalized to [0,1]):**

| Component | Weight | Source |
|-----------|--------|--------|
| EPD rate on broker's loans (trailing 12m) | 30% | KG: loans + delinquency data |
| AVM gap median for broker's appraiser network | 20% | KG: `Appraiser.avmGapMedian` |
| Fraud signal count (trailing 24m) | 25% | KG: `FraudSignal` nodes linked to broker |
| Network centrality in fraud community graph | 15% | GDS: PageRank in broker-appraiser projection |
| Prior FraudCase involvement | 10% | KG: `FraudCase` nodes |

```cypher
// Write updated risk score
MATCH (b:Broker {nmlsId: $nmlsId})
SET b.riskScore          = $newScore,
    b.riskScoreUpdatedAt = datetime(),
    b.riskScoreVersion   = $cycleDate
```

**Thresholds:**

| Score Range | Action |
|-------------|--------|
| 0.0 – 0.40 | No action |
| 0.40 – 0.65 | Add to enhanced monitoring list |
| 0.65 – 0.80 | Auto-escalate to enhanced due diligence on next tape |
| > 0.80 | Create `FraudCase {requiresHumanReview: true}`, block automatic tape approval |

---

### Module 6 — Cross-Deal Pattern Matching (New Tape Context)

When a new tape is ingested, enrich it with cross-deal intelligence:

```cypher
// For each broker in the new tape, retrieve their full network context
MATCH (b:Broker)-[:BROKERED_BY]-(l:Loan)
WHERE b.nmlsId IN $tapebrokerNmlsIds
OPTIONAL MATCH (b)-[:CO_OCCURRED_WITH]-(a:Appraiser)
OPTIONAL MATCH (b)-[:HAS_FRAUD_SIGNAL]->(fs:FraudSignal)
RETURN
  b.nmlsId,
  b.riskScore,
  count(DISTINCT l)  AS priorLoanCount,
  count(DISTINCT a)  AS appraisersWorkedWith,
  count(DISTINCT fs) AS priorFraudSignals
```

Surface this context in the due diligence report for the new tape. The analyst sees:
- Which brokers in this pool have elevated risk scores
- Which broker-appraiser pairs in this pool have prior co-occurrence history
- Any brokers with open fraud cases in the KG

---

## Output

```python
class FraudDetectionResult(BaseModel):
    cycle_date: date
    scope: Literal["TAPE_INGESTION", "MONTHLY_CYCLE", "INVESTIGATOR_REQUEST"]
    tape_id: str | None          # Set only for TAPE_INGESTION scope
    signals_created: int
    cases_opened: int
    broker_scores_updated: int
    high_risk_brokers: list[BrokerRiskSummary]   # Score > 0.65
    cases_requiring_review: list[FraudCaseSummary]
    community_alerts: list[CommunityAlert]        # Louvain communities flagged
```

---

## Knowledge Graph Writes Summary

| Write Operation | Node/Rel | Trigger |
|----------------|----------|---------|
| Create `FraudSignal` | `(:FraudSignal)` | Any module detects a signal |
| Link signal to broker | `[:HAS_FRAUD_SIGNAL]` | Signal created |
| Update broker risk score | `(:Broker).riskScore` | Module 5 |
| Create or update co-occurrence | `[:CO_OCCURRED_WITH]` | Module 3 |
| Write Louvain community ID | `(:Broker).louvainCommunityId` | GDS run |
| Create `FraudCase` | `(:FraudCase)` | Evidence threshold met |
| Link signals to case | `[:PART_OF_CASE]` | FraudCase created |
| Queue for human review | `(:FraudCase).requiresHumanReview = true` | Score > 0.80 or threshold met |

All fraud node writes are append-only per `graph-schema-rules.md § Immutability Invariants`.
