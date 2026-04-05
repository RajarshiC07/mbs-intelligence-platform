---
description: >
  Naming conventions, index strategy, Cypher style, and GDS plugin rules for
  the MBS Intelligence Platform Neo4j Knowledge Graph.
applyTo: "**/*.cypher, **/*.py, infrastructure/schema/**"
---

# Neo4j Conventions — MBS Intelligence Platform

These conventions govern every aspect of how we interact with and extend the Neo4j Knowledge Graph.
Follow them precisely. Do not introduce new patterns without updating this file and the domain model.

---

## Node Label Conventions

### Casing
Node labels use **PascalCase**.

```
(:Loan)            ✓
(:BrokerRiskProfile)  ✓
(:loan)            ✗
(:broker_risk_profile)  ✗
```

### Approved Node Labels (as declared in domain model)

| Label | Owned By Context | Key Properties |
|-------|-----------------|----------------|
| `(:Loan)` | Loan Tape Ingestion | `loanId, fico, ltv, cltv, dti, upb, wac, originationDate` |
| `(:LoanTape)` | Loan Tape Ingestion | `tapeId, originatorId, receivedDate, status` |
| `(:Originator)` | Loan Tape Ingestion | `originatorId, name, nmlsId, historicalDefectRate` |
| `(:Broker)` | Fraud Intelligence | `brokerId, nmlsId, name, riskScore, riskScoreUpdatedAt` |
| `(:Appraiser)` | Fraud Intelligence | `appraiserId, licenseNumber, state, avmGapMedian` |
| `(:FraudCase)` | Fraud Intelligence | `caseId, openedAt, status, sarFiled` |
| `(:FraudSignal)` | Fraud Intelligence | `signalId, signalType, confidence, detectedAt` |
| `(:Property)` | Shared Kernel | `propertyId, msaCode, state, zipCode, floodZone` |
| `(:Pool)` | Post-Securitization Monitoring | `poolId, cusip, closingDate, servicerId` |
| `(:EarlyWarningAlert)` | Post-Securitization Monitoring | `alertId, alertType, severity, detectedAt, resolvedAt` |
| `(:MacroCycleSnapshot)` | Post-Securitization Monitoring | `snapshotId, cycleDate, ffr, nationalHpa` |
| `(:MSAMetrics)` | Post-Securitization Monitoring | `msaCode, msaName, hpa1m, hpa3m, hpa12m, unemployment` |

**Before adding a new label**, it must be declared in `memory/mbs-domain-model.md` first.

---

## Relationship Type Conventions

### Casing
Relationship types use **SCREAMING_SNAKE_CASE**.

```
[:ORIGINATED_BY]      ✓
[:HAS_FRAUD_SIGNAL]   ✓
[:originated_by]      ✗
[:OriginatedBy]       ✗
```

### Approved Relationship Types

| Type | From → To | Key Properties | Context |
|------|-----------|----------------|---------|
| `[:ORIGINATED_BY]` | `(:Loan)→(:Originator)` | `originationDate` | Loan Tape Ingestion |
| `[:BROKERED_BY]` | `(:Loan)→(:Broker)` | `originationDate` | Loan Tape Ingestion |
| `[:APPRAISED_BY]` | `(:Loan)→(:Appraiser)` | `appraisalDate, appraisedValue, avmValue` | Fraud Intelligence |
| `[:SECURES]` | `(:Loan)→(:Property)` | `lienPosition` | Shared Kernel |
| `[:BELONGS_TO_POOL]` | `(:Loan)→(:Pool)` | `addedDate` | Post-Securitization Monitoring |
| `[:SERVICES]` | `(:Servicer)→(:Pool)` | `effectiveDate` | Post-Securitization Monitoring |
| `[:HAS_FRAUD_SIGNAL]` | `(:Broker)→(:FraudSignal)` | — | Fraud Intelligence |
| `[:PART_OF_CASE]` | `(:FraudSignal)→(:FraudCase)` | `addedAt` | Fraud Intelligence |
| `[:CO_OCCURRED_WITH]` | `(:Broker)↔(:Appraiser)` | `dealCount, lastSeenDate` | Fraud Intelligence |
| `[:MACRO_CONTEXT]` | `(:Pool)→(:MacroCycleSnapshot)` | `cycleDate` | Post-Securitization Monitoring |
| `[:LOCATED_IN_MSA]` | `(:Property)→(:MSAMetrics)` | — | Shared Kernel |

---

## Property Naming Conventions

- **camelCase** for all node and relationship properties.
- Booleans: prefix with `is` or `has` (e.g., `isFlaggedForReview`, `hasSarFiled`).
- Dates: use ISO 8601 strings (`"2026-03-28"`) or Neo4j `date()` type — never epoch integers.
- Monetary amounts: store in **dollars** (not cents), as `Float`.
- Rates: store as **decimal fractions** (0.05 for 5%), not basis points, unless the property name explicitly says `Bps` (e.g., `spreadBps`).
- Scores: store as `Float` in [0.0, 1.0] range unless explicitly noted otherwise.

### MISMO-Normalized Field Mapping Examples

Raw tape fields must be mapped to MISMO names before persisting:

| Raw (common variants) | MISMO / Platform property |
|-----------------------|--------------------------|
| `FICO`, `credit_score`, `CreditScore` | `fico` |
| `LTV`, `loan_to_value`, `LoanToValue` | `ltv` |
| `UPB`, `unpaid_principal`, `CurrentBalance` | `upb` |
| `DTI`, `debt_to_income`, `DebtToIncome` | `dti` |
| `orig_date`, `OriginationDate` | `originationDate` |
| `broker_id`, `BrokerID`, `NMLS_Broker` | `brokerId` (resolve via NMLS) |

---

## Index Strategy

Declare all indexes in `infrastructure/schema/indexes.cypher`. Every property used in a `MATCH` predicate must be indexed.

### Required Indexes (baseline)

```cypher
// Uniqueness constraints (also create an index)
CREATE CONSTRAINT loan_id_unique IF NOT EXISTS
  FOR (l:Loan) REQUIRE l.loanId IS UNIQUE;

CREATE CONSTRAINT broker_nmls_unique IF NOT EXISTS
  FOR (b:Broker) REQUIRE b.nmlsId IS UNIQUE;

CREATE CONSTRAINT originator_nmls_unique IF NOT EXISTS
  FOR (o:Originator) REQUIRE o.nmlsId IS UNIQUE;

CREATE CONSTRAINT pool_cusip_unique IF NOT EXISTS
  FOR (p:Pool) REQUIRE p.cusip IS UNIQUE;

CREATE CONSTRAINT property_id_unique IF NOT EXISTS
  FOR (pr:Property) REQUIRE pr.propertyId IS UNIQUE;

// Range indexes for numeric lookups
CREATE INDEX loan_fico_index IF NOT EXISTS FOR (l:Loan) ON (l.fico);
CREATE INDEX loan_ltv_index IF NOT EXISTS FOR (l:Loan) ON (l.ltv);
CREATE INDEX loan_upb_index IF NOT EXISTS FOR (l:Loan) ON (l.upb);
CREATE INDEX loan_origination_date_index IF NOT EXISTS FOR (l:Loan) ON (l.originationDate);

// Lookup indexes for fraud queries
CREATE INDEX broker_risk_score_index IF NOT EXISTS FOR (b:Broker) ON (b.riskScore);
CREATE INDEX fraud_signal_type_index IF NOT EXISTS FOR (f:FraudSignal) ON (f.signalType);
CREATE INDEX alert_severity_index IF NOT EXISTS FOR (a:EarlyWarningAlert) ON (a.severity);

// Geographic
CREATE INDEX property_msa_index IF NOT EXISTS FOR (pr:Property) ON (pr.msaCode);
CREATE INDEX property_state_index IF NOT EXISTS FOR (pr:Property) ON (pr.state);
```

---

## Cypher Style Guide

### Parameterization (Mandatory)
Always use parameters. Never concatenate values into query strings.

```python
# CORRECT
session.run(
    "MATCH (b:Broker {nmlsId: $nmlsId}) RETURN b",
    nmlsId=broker_nmls_id
)

# WRONG — Cypher injection risk
session.run(f"MATCH (b:Broker {{nmlsId: '{broker_nmls_id}'}}) RETURN b")
```

### MERGE vs CREATE
- Use `MERGE` for entities that may already exist (brokers, originators, properties).
- Use `CREATE` only when you are certain the node does not exist (e.g., a new `FraudCase` with a generated UUID).
- Always provide `ON CREATE SET` and `ON MATCH SET` clauses with `MERGE` to avoid silent no-ops.

```cypher
MERGE (b:Broker {nmlsId: $nmlsId})
ON CREATE SET
  b.brokerId    = $brokerId,
  b.name        = $name,
  b.riskScore   = 0.0,
  b.createdAt   = datetime()
ON MATCH SET
  b.name        = $name,
  b.updatedAt   = datetime()
```

### Formatting
- One clause per line (MATCH, WHERE, WITH, RETURN each on its own line).
- Indent continuation lines by 2 spaces.
- Alias all return values explicitly — do not return bare node variables in production queries.

```cypher
MATCH (l:Loan)-[:BROKERED_BY]->(b:Broker)
WHERE b.riskScore > $threshold
  AND l.originationDate >= $cutoffDate
WITH b, count(l) AS loanCount, avg(l.ltv) AS avgLtv
RETURN
  b.nmlsId     AS brokerNmlsId,
  b.riskScore  AS riskScore,
  loanCount,
  avgLtv
ORDER BY b.riskScore DESC
```

---

## GDS Plugin Conventions

### When to Use GDS
GDS (Graph Data Science) is used for:
- Broker-Appraiser co-occurrence community detection (Louvain)
- Fraud ring identification (Weakly Connected Components)
- Broker centrality scoring (PageRank within deal network)
- Pool similarity scoring (node embedding similarity)

### GDS Graph Projection Naming

Named graph projections follow the pattern: `{context}-{purpose}-{date}`

Examples:
- `fraud-broker-appraiser-2026-04`
- `monitoring-pool-similarity-2026-04`

Projections are **temporary** — drop them after algorithm execution. Never leave named projections persisted in production.

```cypher
// Drop after use
CALL gds.graph.drop('fraud-broker-appraiser-2026-04') YIELD graphName;
```

### GDS Result Persistence
Write GDS algorithm results back to the graph immediately as node properties:

```cypher
CALL gds.pageRank.write('fraud-broker-appraiser-2026-04', {
  writeProperty: 'networkCentrality',
  maxIterations: 20
})
```

---

## Fraud Audit Trail — Immutability Rules

All writes to fraud-related nodes (`FraudSignal`, `FraudCase`, audit relationships) are **append-only**:

- Never `SET` a property on an existing `FraudSignal` or `FraudCase` node.
- To record a status change, create a new `(:FraudCaseEvent)` node and link it: `[:HAS_EVENT]`.
- Never `DELETE` or `DETACH DELETE` fraud-related nodes.
- Every fraud-related write must carry a `createdAt` timestamp and `createdBy` (agent or user ID).

```cypher
// CORRECT — append a new event
CREATE (e:FraudCaseEvent {
  eventId:   $eventId,
  caseId:    $caseId,
  eventType: 'STATUS_CHANGED',
  newStatus: 'ESCALATED_TO_LEGAL',
  createdAt: datetime(),
  createdBy: $agentId
})
WITH e
MATCH (c:FraudCase {caseId: $caseId})
CREATE (c)-[:HAS_EVENT]->(e)

// WRONG — mutating existing case node
MATCH (c:FraudCase {caseId: $caseId})
SET c.status = 'ESCALATED_TO_LEGAL'   ← Never do this
```
