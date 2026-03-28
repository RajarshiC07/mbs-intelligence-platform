# MBS Intelligence Platform

> A Neo4j knowledge graph platform that augments every phase of the mortgage-backed securities lifecycle — from loan tape ingestion through post-securitization monitoring — with graph-based intelligence, agentic AI, and explainable fraud detection.

---

## Overview

The mortgage-backed securities (MBS) industry still operates largely on spreadsheets, email chains, and institutional memory stored in individual analysts' heads. This platform replaces that fragility with a persistent, connected knowledge graph that accumulates intelligence across every deal the Firm has ever touched.

The system is not a replacement for existing workflows — it is an augmentation layer. Every phase of the MBS lifecycle, from acquiring a loan tape to distributing principal and interest to investors, continues to operate as it does today. The platform adds a layer of structured intelligence at each phase: automating what can be automated, surfacing patterns that humans cannot see manually, and keeping humans in the loop for decisions that require judgment.

---

## What This Platform Does

### Pre-Securitization

| Capability | What It Solves |
|---|---|
| **Automated Loan Tape Ingestion** | Eliminates manual spreadsheet triage; normalizes all tapes to MISMO standard on arrival |
| **AI Document Reading (HITL)** | Cross-references tape values against source documents (W-2s, appraisals, bank statements); discrepancies flagged for human review |
| **Institutional Memory** | Every originator and broker the Firm has dealt with is remembered — historical quality metrics surface automatically on each new tape |
| **Pre-Securitization Pool Optimization** | Graph-query-driven pool construction that satisfies multi-dimensional rating agency constraints; replaces Excel solver approaches |
| **Stress Test Simulation** | Monte Carlo scenarios (rate shock, HPA decline, recession) run against candidate pools before deal close |

### Post-Securitization

| Capability | What It Solves |
|---|---|
| **Monthly Knowledge Graph Refresh** | Servicer remittance data updates all loan nodes monthly; no manual MIS processing |
| **Early Warning System** | Pools approaching stress thresholds (delinquency migration, prepayment speed deviation) trigger proactive alerts before losses materialize |
| **Investor Mandate Matching** | Sales desk support tool that generates ranked investor call lists based on tranche-mandate fit; replaces memory-based matching |

### Cross-Lifecycle

| Capability | What It Solves |
|---|---|
| **Broker Fraud Detection** | Graph pattern matching detects hub-and-spoke rings, appraisal fraud clusters, and straw buyer chains across all deals — not just the current one |
| **Macro & Geopolitical Intelligence** | Agentic AI fetches macro data (FRED, BLS, FHFA HPI, CFPB, FEMA, Bloomberg) monthly and stores it as structured context nodes in the graph |
| **Explainable Risk Scoring** | Every risk flag includes a graph-path explanation ("this broker appears in 12 pools with above-average delinquency") — designed for SR 11-7 model risk governance compliance |

---

## Lifecycle Coverage

This platform covers 9 phases of the MBS lifecycle:

1. **Loan Tape Acquisition** — automated ingestion, validation, originator enrichment
2. **Due Diligence** — AI document reading, HITL review routing, broker background automation
3. **Pool Formation & Pre-Securitization Optimization** — graph-driven pool construction, constraint satisfaction, stress testing
4. **Rating Agency Process & Deal Structuring** — scenario modeling, tranche simulation, stress test documentation support
5. **Investor Marketing & Book Building** — mandate-matching recommendation engine, suitability pre-screening
6. **Deal Closing & Trust Formation** — final pool validation, PSA/REMIC compliance checks
7. **Post-Securitization Monitoring & Early Warning** — monthly tape refresh, delinquency trend detection, portfolio alerts
8. **Fraud Detection** — continuous broker network analysis, entity resolution, anomaly scoring
9. **Macro & Geopolitical Intelligence** — agentic macro data integration, event-driven pool re-scoring

---

## Why a Knowledge Graph?

The relationships in this domain are the signal. A loan is not risky in isolation — it is risky because its broker submitted 40 similar loans across 12 deals and 30% of them are now delinquent. A pool is not optimized in isolation — it is optimized in the context of rating agency concentration constraints, geographic diversification, and the Firm's historical loss experience with similar collateral.

These multi-hop relationships are expensive or impossible to query in relational systems. Neo4j models them natively. The graph accumulates intelligence across every deal over time, creating a compounding institutional memory that no individual analyst or spreadsheet can replicate.

---

## Key Design Principles

- **No post-securitization loan substitution** — once a trust is formed under a PSA/REMIC structure, loans are locked. All optimization occurs pre-securitization. This is a hard legal constraint, not a design choice.
- **Human in the loop** — AI flags, extracts, and scores. Humans make final decisions on fraud blacklisting, document exceptions, and investor suitability approvals.
- **Monthly batch cadence** — loan tape data is inherently monthly. The system is designed around this reality, not against it.
- **SR 11-7 compliant by design** — all predictive models use explainable methods (gradient boosting + SHAP, graph attention networks) with full governance documentation from day one.

---

## Project Status

| Phase | Scope | Status |
|---|---|---|
| **Phase 1: Foundation** | Loan tape ingestion pipeline, Neo4j schema, graph population, data quality dashboards | Planning |
| **Phase 2: Analytics** | Broker fraud detection, delinquency risk scoring, pool-level stress indicators | Not Started |
| **Phase 3: Optimization** | Pre-securitization pool optimizer, investor recommendation engine | Not Started |
| **Phase 4: Intelligence** | Agentic AI workflows, macro data integration, predictive early warning system | Not Started |

---

## Documentation

| Document | Description |
|---|---|
| [`docs/MBS_Platform_Planning_Document.md`](docs/MBS_Platform_Planning_Document.md) | Full business planning document — 9 lifecycle phases, current state, key terms, pain points, and proposed changes for each phase |

Architecture design document — in progress.

---

## Technology Stack (Planned)

| Layer | Technology |
|---|---|
| Knowledge Graph | Neo4j (AuraDB or self-hosted) |
| Agentic AI | LlamaIndex multi-agent framework |
| LLM | Claude API (Anthropic) |
| Data Ingestion | Python, Pandas, MISMO schema validation |
| Macro Data Sources | FRED, BLS, FHFA HPI, CFPB, FEMA, Bloomberg |
| Graph Algorithms | Neo4j GDS (Graph Data Science library) |

---

## Domain Glossary

| Term | Definition |
|---|---|
| MBS | Mortgage-Backed Security — a bond collateralized by a pool of mortgage loans |
| REMIC | Real Estate Mortgage Investment Conduit — the tax structure used by most MBS trusts |
| PSA | Pooling and Servicing Agreement — the legal contract governing a securitization trust |
| Loan Tape | The master data file containing all loan-level attributes for a pool |
| MISMO | Mortgage Industry Standards Maintenance Organization — defines standard field formats |
| WAC / WAM / WALTV | Weighted Average Coupon / Maturity / LTV — key pool-level metrics |
| LTV | Loan-to-Value ratio — loan balance divided by property appraised value |
| DTI | Debt-to-Income ratio — borrower's monthly debt payments divided by gross income |
| DSCR | Debt Service Coverage Ratio — used for investment property loans |
| TPDD | Third-Party Due Diligence — specialist firm hired to verify loan documentation |
| SR 11-7 | Federal Reserve model risk management guidance — governs predictive model deployment |
| HITL | Human-in-the-Loop — AI flags and extracts; human makes the final decision |
| HPA | Home Price Appreciation — rate of change in property values in a given market |
| GDS | Graph Data Science — Neo4j's library of graph algorithms |

Full glossary in [`docs/MBS_Platform_Planning_Document.md`](docs/MBS_Platform_Planning_Document.md).

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

*This is a personal research and portfolio project. It does not use or contain any proprietary, confidential, or non-public data from any financial institution.*
