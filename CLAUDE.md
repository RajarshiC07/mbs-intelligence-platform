# MBS Intelligence Platform — Claude Instructions

## Project Context

This repository is the **MBS Intelligence Platform** — a Neo4j Knowledge Graph + LlamaIndex agentic AI system that augments every phase of the Mortgage-Backed Securities lifecycle at a major dealer bank (Goldman Sachs, JP Morgan, or equivalent). The platform does **not** replace existing deal infrastructure; it adds a persistent intelligence layer that currently does not exist.

**Primary focus phases for active development: 1, 2, 7, 8, 9.**

---

## What We Are Building

| Phase | Name | Our Intervention |
|-------|------|-----------------|
| **1** | Loan Tape Acquisition | Automated ingestion pipeline, MISMO normalization, originator institutional memory |
| **2** | Due Diligence | AI Document Reader (HITL), 100% loan statistical screening, cross-deal fraud pattern detection |
| **7** | Post-Securitization Monitoring | Proactive alerting, cross-pool pattern recognition, Early Warning System |
| **8** | Fraud Detection (Cross-Cutting) | Persistent Fraud Intelligence Network, Broker Risk Score, cross-deal network analysis |
| **9** | Macro & Geopolitical Intelligence | Monthly AI agent cycle, MSA-level HPA tracking, pool-level macro exposure scoring |

Phases 3–6 are defined in the planning document but are **not in active development scope** — do not extend them unless explicitly asked.

---

## Architecture Principles

### Knowledge Graph First
- Neo4j is the system of record for all entities: loans, originators, brokers, appraisers, properties, servicers, pools, macroeconomic snapshots.
- **No new Neo4j node label or relationship type may be introduced in code before it is declared in the domain model** (`memory/mbs-domain-model.md`).
- The domain model leads; implementation follows.

### Human-in-the-Loop (HITL) — Non-Negotiable
The following decisions **always require human approval** before the system acts. Never write code that bypasses these gates:
- Blacklisting a broker or appraiser
- Filing or recommending a SAR (Suspicious Activity Report) to FinCEN
- Escalating a fraud case to legal review
- Any write to the `FraudCase` aggregate

### Bounded Contexts
Four bounded contexts own the domain. Do not mix concerns across context boundaries:

| Context | Owning Module | Key Aggregates |
|---------|--------------|----------------|
| Loan Tape Ingestion | `loan_tape_pipeline` | `LoanTape`, `Loan`, `Originator` |
| Fraud Intelligence | `fraud_intelligence_network` | `BrokerRiskProfile`, `FraudCase`, `Appraiser` |
| Post-Securitization Monitoring | `early_warning_engine` | `Pool`, `EarlyWarningAlert`, `MacroCycleSnapshot` |
| Knowledge Graph | `neo4j_kg_service` | `GraphNode`, `GraphRelationship` |

### MISMO Normalization
All loan tape fields must be normalized to MISMO standards before any downstream processing. Raw field names from the originator tape are never persisted as graph properties — only normalized names.

---

## Tech Stack

- **Graph DB:** Neo4j (community or enterprise) with GDS plugin for graph algorithms
- **Orchestration:** Python, LlamaIndex agentic workflows
- **AI:** Claude API (claude-sonnet-4-6 for most tasks; claude-opus-4-6 for complex reasoning)
- **Data Sources:** FRED (interest rates), FHFA HPI (home prices), Freddie Mac loan performance, CFPB HMDA (originator profiling), FEMA flood zones
- **Validation:** Pydantic models for all loan tape schemas

---

## Coding Standards

### General
- Python 3.11+. Type hints required on all public functions.
- Pydantic v2 for all data models — no raw dicts crossing module boundaries.
- Never commit loan data, borrower PII, credentials, or `.env` files. See `.gitignore`.
- Tests must hit real Neo4j (test container or dedicated test DB). Do not mock graph queries.

### Cypher
- Use parameterized queries — never string-concatenate user data into Cypher. SQL-injection-equivalent risk applies to Cypher.
- Index all properties used in `MATCH` predicates. Declare indexes in `infrastructure/schema/`.
- Follow conventions in `.claude/rules/neo4j-conventions.md`.
- Follow schema rules in `.claude/rules/graph-schema-rules.md`.

### AI Agents
- Every LlamaIndex agent that can write to the KG must pass through a validation gate before executing writes.
- Fraud-related writes are append-only — never update or delete `FraudCase`, `FraudSignal`, or audit trail nodes.
- Agent definitions live in `.claude/agents/`.

### Domain Model
- All structural changes to `memory/mbs-domain-model.md` follow the rules in `.claude/rules/domain-model-standards.md`.
- Changes to bounded contexts, shared kernel, or cross-cutting concerns require an ADR in `architecture/adr/`.

---

## Data Sensitivity Rules

- **Never** read, print, log, or persist borrower PII (names, SSNs, addresses) beyond what is structurally required.
- Loan-level data in tests must be synthetic — never use real loan tapes.
- Property addresses in the graph use `propertyId` (hashed), not raw street addresses.
- All fraud audit trail entries are immutable once written. Write a new node; never update an existing one.

---

## Key Domain Vocabulary (Quick Reference)

| Term | Meaning |
|------|---------|
| UPB | Unpaid Principal Balance |
| LTV | Loan-to-Value ratio |
| CLTV | Combined LTV (all liens) |
| DTI | Debt-to-Income ratio |
| FICO | Borrower credit score (300–850) |
| WAC | Weighted Average Coupon |
| WAM | Weighted Average Maturity |
| CDR | Constant Default Rate |
| CPR | Constant Prepayment Rate |
| EPD | Early Payment Default (default within first 3 payments — severe fraud signal) |
| HPA | Home Price Appreciation |
| MSA | Metropolitan Statistical Area |
| TPDD | Third-Party Due Diligence firm |
| SAR | Suspicious Activity Report (FinCEN filing) |
| MISMO | Mortgage Industry Standards Maintenance Organization |
| NMLS | Nationwide Mortgage Licensing System (broker/originator IDs) |

Full vocabulary is in `memory/mbs-domain-model.md § Ubiquitous Language`.

---

## Specialized Rules & Agents

| File | Purpose |
|------|---------|
| `.claude/rules/domain-model-standards.md` | Structural rules for `memory/mbs-domain-model.md` |
| `.claude/rules/neo4j-conventions.md` | Node labels, rel types, property naming, index strategy |
| `.claude/rules/graph-schema-rules.md` | Schema governance, immutability, cross-context constraints |
| `.claude/agents/loan-validator.md` | Sub-agent: loan tape field validation and MISMO mapping |
| `.claude/agents/fraud-signal-detector.md` | Sub-agent: cross-deal fraud pattern detection |
| `.claude/commands/pool-analysis.md` | `/pool-analysis` command definition |

---

## Planning Document

The full business context, phase-by-phase current state, key terms, pain points, and proposals are in:
`docs/MBS_Platform_Planning_Document.md`

Read this before proposing architectural changes or new features.
