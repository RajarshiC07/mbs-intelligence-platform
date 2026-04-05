# /pool-analysis

Run a comprehensive pool analysis for a specified pool or loan tape.
Covers pool stratification, macro exposure scoring, fraud signal summary,
early warning status, and cross-vintage comparison.

## Usage

```
/pool-analysis [pool-id or tape-id] [--phase <1|7|8|9|all>] [--format <report|json|table>]
```

**Arguments:**
- `pool-id` — CUSIP or internal pool ID (post-securitization, Phase 7+)
- `tape-id` — Tape ID (pre-securitization, Phases 1–2)
- `--phase` — Limit analysis to a specific phase's lens (default: `all`)
- `--format` — Output format (default: `report`)

**Examples:**
```
/pool-analysis POOL-2026-001
/pool-analysis TAPE-2026-042 --phase 1
/pool-analysis CUSIP-12345678 --phase 7 --format table
```

---

## What This Command Does

When invoked, perform the following analysis steps. Show progress as each step completes.

---

### Step 1 — Resolve Pool/Tape

Determine if the argument is a tape ID or pool ID by querying the KG:

```cypher
// Try tape first
MATCH (t:LoanTape {tapeId: $id}) RETURN t, 'TAPE' AS type
UNION
MATCH (p:Pool {poolId: $id}) RETURN p, 'POOL' AS type
UNION
MATCH (p:Pool {cusip: $id}) RETURN p, 'POOL' AS type
```

If not found, report clearly: `Pool/tape [id] not found in Knowledge Graph.`

---

### Step 2 — Pool Stratification (Phase 1 / Phase 2 lens)

Compute the standard pool stratification table across key risk dimensions.
Display as a formatted table:

**FICO Distribution**
| Bucket | Count | % Count | % UPB |
|--------|-------|---------|-------|
| < 620 | | | |
| 620–679 | | | |
| 680–739 | | | |
| 740–799 | | | |
| ≥ 800 | | | |

**LTV Distribution** (buckets: < 60%, 60–70%, 70–80%, 80–90%, 90–95%, > 95%)

**Geographic Concentration** (top 10 states by UPB %)

**Product Type Mix** (Full Doc / Alt-A / DSCR)

**Occupancy Mix** (Primary / Second Home / Investment)

**Loan Purpose Mix** (Purchase / Rate-Term Refi / Cash-Out Refi)

**Pool Summary Statistics:**
- Total UPB, Loan Count, WAC, WAM, WALTV, Weighted Average FICO, Weighted Average DTI

Flag any concentration that triggers a `GEOGRAPHIC_CONCENTRATION` signal (> 20% UPB in single MSA).

---

### Step 3 — Fraud Signal Summary (Phase 8 lens)

Query all fraud signals associated with loans in this pool:

```cypher
MATCH (l:Loan)-[:BELONGS_TO_POOL]->(p:Pool {poolId: $poolId})
OPTIONAL MATCH (b:Broker)<-[:BROKERED_BY]-(l)
OPTIONAL MATCH (b)-[:HAS_FRAUD_SIGNAL]->(fs:FraudSignal)
RETURN
  fs.signalType,
  count(fs)     AS signalCount,
  avg(b.riskScore) AS avgBrokerRiskScore
```

Display:
- Table of fraud signal types and counts
- Number of brokers with risk score > 0.65
- Any open FraudCases linked to brokers in this pool
- Broker-appraiser co-occurrence flags (CO_OCCURRED_WITH relationships in pool)

---

### Step 4 — Early Warning Status (Phase 7 lens, Pool only)

For post-securitization pools only. Query current performance and active alerts:

```cypher
MATCH (p:Pool {poolId: $poolId})
OPTIONAL MATCH (p)<-[:BELONGS_TO_POOL]-(l:Loan)
OPTIONAL MATCH (a:EarlyWarningAlert {poolId: $poolId, resolvedAt: null})
RETURN
  p.poolId,
  count(l)                                        AS loanCount,
  avg(CASE WHEN l.delinquencyBucket = '30' THEN 1 ELSE 0 END) AS dq30Rate,
  avg(CASE WHEN l.delinquencyBucket = '60' THEN 1 ELSE 0 END) AS dq60Rate,
  avg(CASE WHEN l.delinquencyBucket = '90' THEN 1 ELSE 0 END) AS dq90Rate,
  collect(DISTINCT a.alertType)                   AS activeAlerts
```

Display:
- Current delinquency bucket distribution
- Active (unresolved) early warning alerts with severity
- Month-over-month DQ rate trend (if prior cycle data available)
- OC test status (if data available)

---

### Step 5 — Macro Exposure Profile (Phase 9 lens)

Query the pool's geographic distribution against current MSA-level macro data:

```cypher
MATCH (l:Loan)-[:BELONGS_TO_POOL]->(p:Pool {poolId: $poolId})
MATCH (l)-[:SECURES]->(pr:Property)-[:LOCATED_IN_MSA]->(m:MSAMetrics)
WITH m.msaCode AS msaCode, m.msaName AS msaName,
     sum(l.upb) AS msaUpb,
     m.hpa1m AS hpa1m, m.hpa3m AS hpa3m, m.unemployment AS unemployment
ORDER BY msaUpb DESC
LIMIT 10
RETURN msaCode, msaName, msaUpb, hpa1m, hpa3m, unemployment
```

Display top-10 MSA exposures with:
- UPB concentration %
- 1-month and 3-month HPA
- Current unemployment rate
- Flag any MSA where pool has > 10% UPB AND hpa1m < -2% → `MACRO_HPA_DECLINE` risk

Fetch the most recent `MacroCycleSnapshot` and report:
- Federal Funds Rate
- National HPA trend
- Any FEMA disaster declarations in pool geographies
- Macro risk summary sentence

---

### Step 6 — Vintage Comparison (Phase 7 lens, Pool only)

Compare this pool's performance metrics against same-vintage peers in the KG:

```cypher
MATCH (p:Pool {poolId: $poolId})
WITH p.closingDate.year AS vintage
MATCH (peer:Pool)
WHERE peer.closingDate.year = vintage
  AND peer.poolId <> $poolId
RETURN
  peer.poolId,
  peer.cusip,
  avg(peer.currentDq30Rate)  AS peerAvgDq30,
  avg(peer.currentCdr)       AS peerAvgCdr
```

Display:
- This pool's CDR and DQ30 vs. vintage peer average
- Percentile rank within vintage cohort
- Flag if pool CDR > 1.5x vintage average → `VINTAGE_UNDERPERFORMANCE` alert

---

### Step 7 — Recommended Actions

Based on findings from all steps, provide a prioritized action list:

```
## Recommended Actions

### Immediate (act within 24 hours)
- [only if CRITICAL alerts or fraud signals above threshold]

### This Week
- [HIGH severity items]

### Routine Monitoring
- [MEDIUM items, next monthly cycle]
```

If no issues found: `Pool [id] is within normal parameters. No immediate action required.`

---

## Report Format Example

```
═══════════════════════════════════════════════════════
 MBS POOL ANALYSIS — POOL-2026-001
 As of: 2026-04-05  |  Scope: All Phases
═══════════════════════════════════════════════════════

📊 POOL STRATIFICATION
  UPB:          $487,234,000   Loan Count: 1,847
  WAC:          6.84%          WAM:        342 months
  WALTV:        0.74           WA FICO:    724
  WA DTI:       0.38

  ⚠  Geographic Concentration: CA = 23.4% UPB (threshold: 20%)

🔍 FRAUD SIGNALS
  AVM_GAP:      14 signals     Brokers > 0.65 risk score: 3
  EPD:          2 signals      Open Fraud Cases: 1

🚨 EARLY WARNINGS (2 active)
  HIGH   — DELINQUENCY_RATE_SPIKE (30-day DQ +180bps MoM)
  MEDIUM — MACRO_HPA_DECLINE (Phoenix MSA -2.3% 1m HPA, 18% pool UPB)

🌍 MACRO EXPOSURE
  Top MSA Concentration: Los Angeles (19.2%), Phoenix (18.1%), Seattle (11.4%)
  FFR: 4.75%  |  National HPA 1m: -0.4%  |  FEMA Declarations: None in pool

📈 VINTAGE COMPARISON (2025 Vintage)
  Pool CDR: 1.8%  |  Vintage Avg CDR: 1.2%  → 50th→85th percentile ⚠

## Recommended Actions
### Immediate
  - Review open FraudCase FC-2026-0033 (Broker NMLS 887234)
### This Week
  - Investigate DQ rate spike — check servicer report for Feb 2026
  - Phoenix MSA HPA decline: assess modification reserve adequacy
═══════════════════════════════════════════════════════
```
