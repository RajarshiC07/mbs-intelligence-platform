# MBS Intelligence Platform — Business Planning Document
### Lifecycle Coverage, Current Operations & Proposed Innovation

**Version:** 2.0
**Date:** 2026-03-28
**Status:** Active Planning

> **Note on Firm & Investor References:** This document is written for generic applicability to major MBS dealer banks (e.g., Goldman Sachs, JP Morgan, Morgan Stanley, Deutsche Bank, Wells Fargo Securities) — collectively referred to as **"the Firm"** throughout. Investor examples are referenced by institution type (Insurance Co., Pension Fund, Hedge Fund, Mutual Fund) rather than by specific firm name.

---

## How to Read This Document

The MBS lifecycle has seven distinct phases from the moment a loan pool is acquired to the point where investors receive their returns. Our platform does not replace any of these phases — it augments each one with intelligence that currently does not exist. For every phase, this document covers:

- **Current State** — how the industry operates today
- **Key Terms** — vocabulary, abbreviations, and concepts specific to this phase
- **Pain Points** — where the current process fails or underperforms
- **Our Proposal** — what we change, and why it matters

---

## Phase 1: Loan Tape Acquisition

### Current State

The lifecycle begins when the Firm agrees to purchase a pool of mortgage loans from an originator, bank, or aggregator. The seller delivers the loan data in the form of a **loan tape** — a structured data file, typically a large spreadsheet or delimited text file, that contains one row per loan and hundreds of columns describing every attribute of that loan.

At this stage, the Firm's team receives the file, opens it, and begins a manual triage: checking whether the fields are populated, whether the data looks credible at a glance, and whether the tape conforms to expected formats. This process is performed by analysts who work off a checklist. There is no automated ingestion. The tape is often renamed, copied into shared drives, and passed between teams via email. Version control is informal — it is common for multiple versions of the same tape to exist simultaneously across different teams.

The quality of loan tapes varies enormously between sellers. Some originators follow industry standards precisely; others deliver files with inconsistent field names, missing data, non-standard date formats, and values that do not align with their own documentation elsewhere in the file.

### Key Terms

| Term | Definition |
|------|-----------|
| **Loan Tape** | The master data file delivered by a seller containing all loan-level attributes for every loan in the pool being sold |
| **Originator** | The bank, credit union, or non-bank lender that created the mortgage loans |
| **Aggregator** | A firm that buys loans from multiple originators and bundles them for onward sale to dealer banks |
| **MISMO** | Mortgage Industry Standards Maintenance Organization — the body that defines standard field names and formats for mortgage data exchange |
| **UPB** | Unpaid Principal Balance — the current outstanding loan amount |
| **LTV** | Loan-to-Value ratio — the loan balance divided by the property's appraised value. A key measure of collateral risk |
| **CLTV** | Combined LTV — accounts for all liens on the property, not just the first mortgage |
| **DTI** | Debt-to-Income ratio — the borrower's monthly debt obligations divided by monthly gross income. A measure of repayment capacity |
| **FICO Score** | Credit score from Fair Isaac Corporation, ranging 300–850. The primary measure of borrower creditworthiness |
| **WAC** | Weighted Average Coupon — the average interest rate across all loans in the pool, weighted by balance |
| **WAM** | Weighted Average Maturity — the average time to maturity across loans, weighted by balance |
| **WALTV** | Weighted Average LTV — the average LTV across loans, weighted by balance |
| **Documentation Type** | How borrower income and assets were verified: Full Doc (W-2/tax returns), Alt-A (limited verification), DSCR (investment properties — no income verification, cash flow based) |
| **Occupancy Status** | Whether the property is owner-occupied (primary residence), second home, or investment property |
| **Loan Purpose** | Purchase (new acquisition), Rate/Term Refinance (lower rate or term), or Cash-Out Refinance (borrower extracting equity) |

### Pain Points

- No systematic validation — errors and gaps in the tape are discovered late, sometimes after pricing decisions have already been made
- No institutional memory — each tape is treated in isolation; there is no comparison to prior tapes from the same originator to identify deteriorating quality
- Version proliferation — multiple tape versions float across teams with no single authoritative source of record
- Broker and originator due diligence is entirely manual — an analyst has to look up the originator or broker separately, with no connection to historical performance data from prior deals

### Our Proposal

We introduce an automated **Loan Tape Ingestion Pipeline** that becomes the single point of entry for all incoming tapes. Every tape submitted flows through the same process: field normalization to MISMO standards, logical consistency checks, statistical validation, and automatic enrichment from the Knowledge Graph's existing memory of every originator and broker the Firm has encountered before.

The key business change is the introduction of **institutional memory**. When a tape arrives from an originator the Firm has dealt with before, the system immediately surfaces that originator's historical quality metrics — their average defect rate, the delinquency performance of prior pools, any fraud flags from past deals. This context, which today lives only in the heads of experienced analysts, becomes a standardized input to every new deal.

---

## Phase 2: Due Diligence

### Current State

Due diligence is the process of verifying that the loans in the tape are what the seller claims them to be. It involves two tracks running in parallel: **data due diligence** (verifying the tape fields against source documents) and **credit due diligence** (assessing whether the loans meet the Firm's credit standards).

For large deals, the Firm typically hires a **third-party due diligence (TPDD) firm** — a specialist firm that physically reviews loan files, verifies borrower income against W-2s and tax returns, confirms property values against appraisal reports, and checks legal documentation. This is labor-intensive: a team of reviewers works through each loan file one at a time. For a pool of 2,000 loans, a full review would be impractical — so the TPDD firm reviews a statistical sample, typically 20-30% of loans, with higher concentration on flagged loans.

Results come back as a **kick-out report**: a list of loans that failed due diligence and the reasons why. The Firm then negotiates with the seller — the seller can cure the defect (provide missing documentation), substitute the loan with another, or accept a price adjustment.

Due diligence for a standard deal takes 30 to 60 days and costs between $500,000 and $2 million in third-party fees, depending on pool size and complexity.

### Key Terms

| Term | Definition |
|------|-----------|
| **Third-Party Due Diligence (TPDD)** | A specialist firm hired to independently verify loan file quality on behalf of the buyer |
| **Kick-Out** | A loan rejected from the pool during due diligence due to a defect — the seller must cure, substitute, or accept a price haircut |
| **Cure** | The seller remedies a defect by providing missing documentation or correcting an error |
| **Substitution** | The seller replaces a kicked-out loan with another loan that meets the agreed criteria |
| **Representations and Warranties (R&W)** | Contractual promises the seller makes about the quality of loans sold. If a loan later defaults and R&Ws were breached, the seller must repurchase it |
| **Repurchase (Put-Back)** | The buyer forces the seller to buy back a loan because R&Ws were found to have been violated |
| **Defect Rate** | The percentage of loans reviewed that had at least one material defect |
| **Material Defect** | A flaw significant enough to affect the loan's credit quality or legal enforceability |
| **AVM** | Automated Valuation Model — a computer-generated property value estimate based on comparable sales and property data. Used to cross-check appraisals |
| **Appraisal** | A formal professional opinion of a property's market value, required at loan origination |
| **Appraisal Inflation** | The practice of overstating a property's value to support a higher loan amount — a common form of origination fraud |
| **Stacking** | A fraud tactic where a borrower obtains multiple mortgage loans on the same property from different lenders simultaneously |
| **Entity Resolution** | The process of determining whether records that appear to be different individuals or properties are actually the same entity |
| **Anomaly Score** | A statistical measure of how unusual a loan's attributes are relative to the rest of the pool |

### Pain Points

- Sample-based review means a large share of loans are never examined — defects in unsampled loans only surface after losses
- No cross-deal fraud detection — a fraudulent broker may appear clean on a single deal review but would be visible if compared across dozens of deals simultaneously
- Appraisal fraud is difficult to detect without a model — a reviewer comparing a single appraisal to an AVM may dismiss a small gap, but a systematic pattern across many loans from the same appraiser is invisible to the reviewer
- The kick-out and negotiation process is entirely email and spreadsheet driven, with no audit trail in a structured system
- Third-party costs are substantial and the timeline is a bottleneck for deal velocity

### Our Proposal

We introduce an **AI-Augmented Due Diligence Layer** that operates on 100% of loans, not a sample. Statistical and relational analysis runs across the entire pool from the moment the tape is received, surfacing the highest-risk loans before any human reviewer opens a single file.

The critical innovation is **cross-deal fraud pattern detection**. By maintaining a persistent Knowledge Graph of every broker, appraiser, property, and borrower the Firm has encountered across all deals over time, we can identify patterns that are invisible at the single-deal level. An appraiser who inflates values by 8% on this deal may have done the same on three prior deals — that pattern is now visible and actionable.

We also introduce an **AI Document Reader** for the verification track. When loan documents (W-2s, bank statements, appraisal reports) are available digitally, the AI reads them, extracts structured values, and compares them against the tape. Discrepancies are flagged automatically and routed to a human analyst — not for full review, but for exception handling. The human reviewer's time is concentrated only where the system has identified a meaningful discrepancy. This is a **Human-in-the-Loop (HITL)** model: AI does the volume work, humans make the final call on exceptions.

---

## Phase 3: Pool Formation & Pre-Securitization Optimization

### Current State

Once due diligence is complete and kicked-out loans are resolved, the Firm's structuring desk assembles the **pool** — the precise set of loans that will collateralize the MBS. Not all loans from the tape necessarily make it into the pool. The structuring desk selects loans based on a combination of credit quality requirements, investor preferences, and rating agency criteria.

Pool formation is a constrained optimization problem: maximize the economics of the deal (yield, pricing) while meeting a set of hard constraints (credit quality floors, geographic concentration limits, product type mix, rating agency requirements). Today, this is done in Excel. Analysts build scenarios — trying different combinations of loans — and evaluate the results. A pool of 2,000 loans from a tape of 5,000 has an effectively infinite combination space; analysts work from rules of thumb and experience, not exhaustive search.

The structuring desk then runs the candidate pool through **rating agency models** — proprietary models used by Moody's, S&P, and Fitch to simulate loan defaults under stress scenarios and determine what credit enhancement is needed for each tranche to achieve a target rating. This process involves back-and-forth with the rating agencies, multiple model runs, and iterative pool adjustments.

The entire process from tape receipt to a finalized pool ready for rating agency submission typically takes 4–8 weeks.

### Key Terms

| Term | Definition |
|------|-----------|
| **Pool** | The specific collection of mortgage loans selected to back an MBS deal |
| **Pool Stratification** | A summary table showing the distribution of key loan attributes (LTV buckets, FICO bands, state concentration, product type) within the pool |
| **Credit Enhancement (CE)** | Structural protection built into the deal to absorb losses before they reach senior investors. Provided by subordinate tranches, overcollateralization, and reserve funds |
| **Tranche** | A slice of the deal with a defined priority of cash flow receipt and risk level. Senior tranches (AAA) are paid first and take last loss; subordinate tranches take first loss |
| **Senior Tranche** | The most protected tranche, typically rated AAA. Receives principal and interest first; absorbs losses last |
| **Subordinate / Junior Tranche** | Lower-rated tranches that absorb losses first. Higher yield but higher risk |
| **Residual / Equity** | The most subordinate piece of the deal — takes all losses first, but receives excess spread after all senior obligations are paid |
| **Overcollateralization (OC)** | The excess of loan collateral value over bond face value — provides a buffer against losses |
| **Excess Spread** | The difference between the interest collected from borrowers and the interest paid to investors. Provides ongoing cushion against losses |
| **Stress Testing** | Simulating pool performance under adverse economic scenarios (house price declines, unemployment spikes, interest rate shocks) to determine how much credit enhancement is needed |
| **CDR** | Constant Default Rate — an annualized rate expressing what percentage of the pool is expected to default |
| **CPR** | Constant Prepayment Rate — an annualized rate expressing how quickly borrowers are paying off their loans early (refinancing or selling) |
| **Severity / Loss Severity** | The percentage of a defaulted loan's balance that is ultimately lost after the property is sold in foreclosure. Typical range: 20–50% |
| **Expected Loss** | CDR × Severity — the anticipated total loss on the pool expressed as a percentage of balance |
| **Geographic Concentration** | The percentage of the pool originating from a single state or metropolitan area — concentration risk if a regional downturn occurs |
| **Vintage** | The year of loan origination. Loans from the same vintage share the same economic conditions at origination |
| **Moody's MILAN / S&P LEVELS** | The proprietary rating agency models used to stress test pools and determine credit enhancement requirements |

### Pain Points

- Excel-based pool construction cannot exhaustively test combination spaces — the optimal pool is never found, only a satisfactory one
- Rating agency model runs are performed manually by analysts, consuming significant time and requiring specialist knowledge
- Stress scenarios are limited to those an analyst thinks to run — systematic scenario coverage is absent
- No learning from prior deals — each pool is constructed from scratch with no institutional knowledge about which loan combinations historically produced better performance

### Our Proposal

We introduce a **Graph-Based Pool Optimizer** that treats pool construction as a systematic, data-driven process rather than a manual one.

The business change is fundamental: instead of an analyst building a pool from intuition and spreadsheet rules, the system searches the loan universe against a defined set of constraints and returns candidate pools ranked by expected performance. The structuring desk's role shifts from building pools to **evaluating and selecting from optimized candidates** — a much higher-value activity.

We also enable **scenario libraries** — pre-built stress scenarios drawn from historical market events (2008 financial crisis, 2020 COVID shock, 1994 rate spike) as well as forward-looking macroeconomic projections. Every candidate pool is automatically evaluated against all scenarios before any rating agency submission, dramatically reducing the number of rounds of back-and-forth required.

The system retains **performance history** from closed deals — which pool characteristics correlated with outperformance or underperformance. Over time this becomes a proprietary feedback loop that no external tool can replicate.

---

## Phase 4: Rating Agency Process & Deal Structuring

### Current State

Before an MBS can be sold to investors, it must receive a credit rating from at least one of the three major rating agencies: Moody's, S&P Global, or Fitch Ratings. Institutional investors — insurance companies, pension funds, banks — are often legally or contractually required to hold rated instruments. The rating determines the price investors will pay, which directly determines the Firm's economics on the deal.

The rating process involves submitting the pool stratification, stress test results, and deal structure documentation to the rating agency. The agency runs its proprietary models, may request additional information, and issues preliminary ratings for each tranche. The Firm then adjusts the deal structure — changing the size and credit enhancement levels of each tranche — to achieve the desired rating profile. This is an iterative process involving multiple rounds of submission and feedback.

Simultaneously, the Firm's structuring desk designs the **waterfall** — the legal rules governing how cash flows from borrower payments are distributed to investors. The waterfall determines the order of payment, what triggers a shift in payment priority (a **trigger event**), and how losses are allocated across tranches.

This phase culminates in the preparation of a **term sheet** and ultimately the **offering memorandum (OM)** or **prospectus** — the legal disclosure document provided to investors before they commit to buying.

### Key Terms

| Term | Definition |
|------|-----------|
| **Rating Agency** | Moody's, S&P Global, or Fitch Ratings — firms that assign credit ratings to debt instruments based on independent analysis of default probability |
| **Credit Rating** | A letter-grade assessment of an instrument's creditworthiness: AAA/Aaa (highest), AA/Aa, A, BBB/Baa (investment grade floor), BB/Ba, B, CCC/Caa (speculative/junk) |
| **Investment Grade** | Rated BBB/Baa or above. Many institutional investors are restricted to investment-grade holdings only |
| **Waterfall** | The contractual rules defining the sequence and priority in which cash flows from the loan pool are distributed to each tranche |
| **Sequential Pay** | A waterfall structure where all principal is directed to the most senior tranche first until it is paid off, then to the next, in sequence |
| **Pro-Rata Pay** | A waterfall structure where principal is distributed proportionally across tranches simultaneously |
| **Trigger Event** | A performance threshold (e.g., delinquency rate exceeding a specified level) that causes the waterfall to switch from pro-rata to sequential payment — protecting senior investors |
| **Overcollateralization (OC) Test** | A regular calculation comparing the value of collateral to the outstanding bond balance. Failing the test redirects cash flow to senior investors |
| **Interest Coverage (IC) Test** | A regular calculation comparing interest collected from the pool to interest owed to investors. Failing triggers the same protective redirect |
| **Term Sheet** | A summary document presented to investors outlining the deal's key economic terms — pool characteristics, tranche sizes, ratings, expected yields |
| **Offering Memorandum (OM) / Prospectus** | The full legal disclosure document for the deal, including risk factors, pool data, waterfall mechanics, and legal structure |
| **Reg AB** | SEC Regulation AB — rules governing the registration and disclosure requirements for asset-backed securities issued publicly in the United States |
| **144A** | A private placement exemption under SEC rules allowing securities to be sold to Qualified Institutional Buyers without full public registration |
| **PSA** | Pooling and Servicing Agreement — the master legal document governing the MBS trust: who services the loans, how cash flows are distributed, and what happens in default |
| **REMIC** | Real Estate Mortgage Investment Conduit — a special tax structure that allows the MBS trust to pass interest income through to investors without being taxed at the trust level. Critical: REMIC status imposes strict rules on what the trust can and cannot do after formation |

### Pain Points

- Rating agency submissions are prepared manually — compiling pool data into agency-specific templates is time-consuming and error-prone
- Multiple rounds of iteration with rating agencies stretch deal timelines by weeks
- Waterfall modeling is done in specialized systems (Intex, Bloomberg) that require licensed software and specialist training
- Disclosure document preparation (OM, prospectus) involves lawyers and compliance teams working from templates updated manually for each deal
- There is no systematic check that pool characteristics disclosed in the OM match the underlying loan tape exactly

### Our Proposal

Our contribution to this phase is primarily **preparation quality and speed**, not replacement of the rating agency process itself.

By the time a pool reaches rating agency submission, it will have been stress-tested against all standard rating agency scenarios through our system. The number of iterative rounds is reduced because the pool arrives optimized. We also propose an **automated disclosure consistency check** — a process that reads the draft OM and verifies that every stated pool characteristic (WAC, WALTV, geographic distribution, FICO distribution) matches the loan-level data in the Knowledge Graph exactly. This eliminates a class of errors that have historically led to regulatory enforcement actions and investor litigation.

Over time, the system builds a **rating agency feedback repository** — a structured record of every pool we have submitted, what the initial rating model output was, and what adjustments were required to reach the final rating. This becomes a training resource for structuring analysts and a basis for improving our pre-submission simulation accuracy.

---

## Phase 5: Investor Marketing & Book Building

### Current State

Once the deal is structured and preliminary ratings are in hand, the Firm's sales and distribution team markets the deal to potential investors. This is the book-building process.

The sales desk maintains relationships with hundreds of institutional investors: insurance companies, pension funds, asset managers, banks, hedge funds, and real estate investment trusts. Each investor type has different risk appetites, regulatory constraints, and mandate requirements. A AAA-rated senior tranche with a 5-year duration appeals to an insurance company seeking safe, duration-matched assets. A BB-rated subordinate tranche with high yield appeals to a hedge fund with a credit strategy.

Today, the sales process relies almost entirely on relationship knowledge that lives in the heads of individual salespeople. A salesperson who has covered insurance companies for ten years knows which of their accounts will buy what type of tranche and at what yield. When that salesperson leaves, that knowledge leaves with them. There is no systematic mapping of investor mandates to available deal inventory.

The process: the sales desk receives the term sheet, holds internal discussions to determine which investors to call, and then the relevant salespeople reach out to their accounts. Investors indicate interest (a "soft circle") which is tracked in email and CRM notes. When enough demand is assembled, the deal is priced and allocations are made.

### Key Terms

| Term | Definition |
|------|-----------|
| **Book Building** | The process of assembling investor demand for a new security before it is priced — investors indicate how much they want to buy and at what yield |
| **Soft Circle** | An investor's non-binding indication of interest in buying a specific tranche at or around the offered price |
| **Oversubscribed** | When investor demand exceeds the available amount of a tranche — gives the Firm pricing power to tighten spread (lower yield) |
| **Undersubscribed** | When investor demand falls short of the tranche size — may require widening spread (higher yield) or reducing tranche size |
| **Spread** | The yield premium above a benchmark rate (typically SOFR) that an investor receives for taking the credit risk of an MBS tranche |
| **SOFR** | Secured Overnight Financing Rate — the primary US dollar interest rate benchmark, replacing LIBOR |
| **Investment Mandate** | A formal document (or set of internal guidelines) defining what types of investments an institution is permitted to hold — rating floors, duration limits, sector restrictions, concentration limits |
| **NAIC** | National Association of Insurance Commissioners — the regulatory body that sets investment guidelines for US insurance companies. Assigns risk-based capital charges to different securities |
| **ERISA** | Employee Retirement Income Security Act — federal law governing pension fund investments. Imposes prudent investor standards and fiduciary duties |
| **Risk-Based Capital (RBC)** | Capital that banks and insurance companies are required to hold against each investment, proportional to its risk. Higher-risk investments require more capital |
| **Duration** | A measure of a bond's sensitivity to interest rate changes, expressed in years. Insurance companies and pension funds typically seek assets that match the duration of their liabilities |
| **MiFID II / Reg BI** | European (MiFID II) and US (Regulation Best Interest) rules requiring that financial products sold to clients are suitable for their needs and risk profile |
| **Allocation** | The final amount of a tranche assigned to each investor after the book is built and the deal is priced |
| **Ticket** | An investor's final confirmed order for a specific amount of a specific tranche |

### Pain Points

- Investor mandate knowledge is undocumented and concentrated in individual salespeople — the Firm has no institutional database of who buys what
- The right investors for a given tranche are identified through personal memory and informal consultation, not systematic analysis
- Regulatory suitability checks (is this tranche permissible for this investor?) are done manually or not at all
- No feedback loop — there is no structured record of which investors passed on which deals and why, which would enable better targeting over time
- Secondary market activity — when investors later trade their positions — generates no intelligence that feeds back into future primary market distribution

### Our Proposal

We introduce a **Mandate-Driven Investor Matching Engine** embedded in the Firm's sales workflow.

The fundamental business change is transforming investor knowledge from tacit (in salespeople's heads) to explicit (in the Knowledge Graph). Every investor's mandate profile is encoded: their minimum rating requirements, duration preferences, sector restrictions, regulatory constraints, and historical purchasing patterns. When a new deal is ready for distribution, the system generates a ranked investor call list for each tranche — ordered by fit score, with the reasoning made transparent.

To be clear: **this does not change the primary market structure**. Investors still do not browse and select MBS freely. The Firm's sales desk still makes the calls and manages the relationships. What changes is that those calls are more targeted, more informed, and more likely to result in a transaction. The salesperson calling an insurance company investor now arrives at that conversation already knowing how the tranche fits the investor's regulatory framework, what similar tranches the investor has bought before, and what the investor has passed on in the past.

We also introduce an **automated suitability pre-check**: before any tranche is offered to any investor, the system verifies that the offering would not violate the investor's known mandate constraints. This reduces regulatory exposure and eliminates wasted conversations.

Over time, the system captures **investor feedback** — not just who bought, but who declined and at what spread — building a proprietary dataset on investor price sensitivity and appetite that makes future book-building progressively more efficient.

---

## Phase 6: Deal Closing & Trust Formation

### Current State

Once the book is built and the deal is priced, the transaction moves to legal closing. This is the most process-intensive phase from a legal and compliance standpoint. The Firm, the trustee, the servicer, rating agencies, and outside counsel all execute a series of agreements and document deliveries simultaneously.

The core legal document executed at closing is the **Pooling and Servicing Agreement (PSA)** — a multi-hundred-page contract that defines every aspect of the trust's operation for its entire life. Once signed, the loans are legally transferred out of the Firm's balance sheet into the trust. The REMIC election is made with the IRS. The trust issues the bonds, which are delivered to investors.

From this point forward, the loans belong to the trust and its investors — the Firm no longer owns them. The Firm's relationship to the deal shifts from principal (owner) to various service roles: the Firm may act as master servicer, paying agent, or administrator, but not as owner of the collateral.

### Key Terms

| Term | Definition |
|------|-----------|
| **Trustee** | A bank appointed to hold the trust assets on behalf of investors and ensure the PSA is followed — typically a major bank's custody division |
| **Closing Date** | The date on which all legal documents are executed, loans are transferred to the trust, and bonds are issued to investors |
| **Cut-Off Date** | The date from which the trust begins collecting cash flows — typically the first of the month of closing |
| **Settlement** | The exchange of bonds for cash — investors deliver payment, the Firm delivers the bonds |
| **DTC** | Depository Trust Company — the central US securities depository through which bond ownership is recorded and settlement occurs |
| **Master Servicer** | The entity responsible for overseeing the collection of borrower payments and their distribution to the trust; often subcontracts day-to-day servicing |
| **Primary Servicer** | The entity with direct contact with borrowers — collecting monthly payments, managing delinquencies, handling modifications and foreclosures |
| **Paying Agent** | The entity responsible for calculating and distributing the monthly interest and principal payments to investors per the waterfall |
| **CUSIP** | A 9-character identifier assigned to each tranche — the universal bond identifier used in trading and settlement systems |
| **Rule 144A / Reg S** | Legal exemptions under which private MBS are sold: 144A to US qualified institutional buyers; Reg S to non-US investors |
| **Legal Final Maturity** | The latest possible date by which all principal on a tranche must be repaid — typically 30+ years, though expected maturity is much shorter |
| **Trustee Report** | A monthly report published by the trustee detailing the pool's performance, cash flow distributions, test results, and trigger status |

### Pain Points

- Closing is document-intensive and largely manual — dozens of documents must be reviewed, signed, and delivered in coordinated sequence
- Data integrity between the loan tape used for deal structuring and the final legal documents has historically been a source of errors
- Post-closing, the Firm's visibility into the pool deteriorates — it becomes dependent on monthly trustee reports rather than having direct access to loan-level data

### Our Proposal

Our primary contribution here is a **closing data integrity check** — an automated comparison between the loan-level data in the Knowledge Graph (as used for structuring) and the final pool documentation submitted for closing. Any discrepancy between what was modeled and what was legally conveyed is flagged before execution.

Post-closing, the Knowledge Graph continues to maintain the pool record. Rather than the Firm's visibility ending at closing, the system links each pool to its ongoing servicer reporting — the Firm retains a live, structured view of pool performance that is absent today.

---

## Phase 7: Post-Securitization Monitoring & Early Warning

### Current State

After closing, the Firm's ongoing relationship with the pool depends on whether it retained a servicing role. If it did, it receives monthly **remittance reports** from the primary servicer — structured data files showing each loan's payment status, any delinquency transitions, prepayments, modifications, and liquidations that occurred during the reporting period.

Portfolio managers and risk teams review these reports to assess the health of pools they are responsible for. The review is almost entirely backward-looking: the report arrives approximately 30 days after the reporting period ends, describing what already happened. By the time a deteriorating trend is identified, losses may already be materializing.

There is no industry-standard early warning system. Monitoring is done differently at every firm — typically by an analyst who opens the monthly report, updates a tracking spreadsheet, and flags anything unusual to a manager. Cross-pool comparison is limited and informal. A deteriorating pool does not automatically generate any notification; it gets noticed only if someone happens to look.

### Key Terms

| Term | Definition |
|------|-----------|
| **Remittance Report** | The monthly report from the servicer to the trustee detailing loan-level payment activity — the primary data source for ongoing pool monitoring |
| **Delinquency Bucket** | The category of a loan based on how many days past due it is: Current, 30-day, 60-day, 90-day, 120+, Foreclosure, REO |
| **30-Day Delinquency Rate** | The percentage of loans that missed at least one payment but have not yet gone 60 days past due — the earliest warning signal |
| **Roll Rate** | The rate at which loans transition from one delinquency bucket to the next — from 30-day to 60-day, 60-day to 90-day, etc. |
| **REO** | Real Estate Owned — property that the trust has taken possession of through foreclosure. An REO property generates no interest income and incurs carrying costs until sold |
| **Modification** | A formal change to a loan's terms (rate reduction, principal deferral, term extension) negotiated with the borrower to avoid foreclosure |
| **Forbearance** | A temporary suspension or reduction of payments granted to a borrower experiencing hardship — the loan is not in default but is not performing normally |
| **Prepayment** | Early payoff of a loan — either the borrower sells the property (housing turnover) or refinances. Prepayment reduces the pool balance and shortens duration |
| **Prepayment Speed (CPR)** | The annualized rate at which loans are prepaying — rising CPR usually signals a falling rate environment (more refinancing) |
| **Default** | A borrower has stopped making payments and the loan has entered the foreclosure process |
| **Loss Severity** | The percentage of the loan's balance that is lost after the property is sold in foreclosure — driven by how much property values have declined |
| **Realized Loss** | An actual loss recorded when a foreclosed property is sold for less than the loan balance — this is the event that consumes credit enhancement |
| **Trigger Event** | A performance threshold breach (e.g., OC test failure, cumulative loss exceeding a threshold) that redirects cash flows to protect senior investors |
| **WAL** | Weighted Average Life — the average time until principal is returned to investors, accounting for both scheduled amortization and prepayments |
| **Extension Risk** | The risk that a pool's WAL extends beyond projections because prepayments slow — typically in rising rate environments |
| **Vintage Analysis** | Comparing pools originated in the same year to assess relative performance and identify systemic trends |

### Pain Points

- Monitoring is reactive — problems are identified after losses have already occurred
- No cross-pool or cross-vintage analysis — each pool is reviewed in isolation, missing patterns that span multiple deals
- Servicer performance varies widely but is not systematically benchmarked — a poorly performing servicer may be handling multiple pools before the problem is identified
- Geopolitical and macroeconomic events (rate shocks, regional economic downturns, natural disasters) that will affect pool performance are not incorporated into monitoring
- No alert system — analysts must proactively check every pool every month to catch deterioration

### Our Proposal

We introduce a **Post-Securitization Early Warning System** that transforms monitoring from reactive to predictive.

The core business change is **proactive alerting**. Rather than analysts reviewing reports and noticing problems, the system monitors every pool continuously and pushes notifications when performance thresholds are approached — before they are breached. A pool whose 30-day delinquency rate has increased by 150 basis points month-over-month generates an alert automatically, regardless of whether an analyst happens to look at it.

The second innovation is **macro-context integration**. Each month, after servicer data is ingested, the system's AI agents pull current macroeconomic data — interest rate movements, regional unemployment, home price appreciation indices, significant geopolitical events — and assess their likely impact on each pool based on its geographic composition and borrower characteristics. A pool concentrated in a region that just experienced a major employer closure, for example, will be automatically flagged for heightened monitoring before the delinquency data reflects the impact.

The third innovation is **cross-pool pattern recognition**. When a performance problem appears in one pool, the system automatically searches for other pools sharing the same servicer, the same originator, the same geographic concentration, or the same borrower profile — surfacing whether the problem is idiosyncratic or systemic.

Human judgment remains central: the system generates alerts and analysis, but portfolio managers make all decisions about escalation and response. Every alert, review, and action is logged back to the Knowledge Graph, creating an audit trail and a dataset for continuous improvement.

---

## Phase 8: Fraud Detection (Cross-Cutting)

### Current State

Mortgage fraud is detected almost exclusively after the fact — when a loan defaults and investigators trace back the origination to discover that documents were falsified, values were inflated, or identities were fabricated. By this point, losses have been incurred. The industry's current approach to fraud prevention is rules-based: loans with certain characteristics (very high LTV, unusual income-to-debt ratios, appraisals far above AVM estimates) are flagged for additional review. These rules catch obvious fraud but miss sophisticated schemes that stay within the rules at the individual loan level while exploiting patterns across many loans.

No firm currently maintains a cross-deal, cross-vintage fraud detection network. Each deal is analyzed in isolation by the due diligence team handling that specific transaction. A fraudulent broker who originated 200 loans across 15 different deals over three years — none of which individually triggered a red flag — is invisible to the current system.

### Key Terms

| Term | Definition |
|------|-----------|
| **Mortgage Fraud** | The intentional misrepresentation of information in a mortgage application or loan document to obtain financing that would not otherwise be approved |
| **Fraud for Housing** | Fraud committed by borrowers misrepresenting their income or assets to qualify for a loan they intend to actually live in — typically smaller scale |
| **Fraud for Profit** | Organized fraud by industry insiders (brokers, appraisers, attorneys) to extract cash from the mortgage system — typically larger scale with multiple co-conspirators |
| **Income Fraud** | Falsification of income documents (W-2s, pay stubs, tax returns) to overstate borrower income and qualify for a larger loan |
| **Appraisal Fraud** | Deliberate inflation of a property's appraised value to support a higher loan amount — often involves collusion between the broker and appraiser |
| **Straw Buyer** | A person who allows their identity to be used to purchase a property on behalf of someone else, typically as part of a fraud scheme |
| **Occupancy Fraud** | Misrepresenting a property as an owner-occupied primary residence (lower rate, lower down payment) when it is actually an investment property |
| **Identity Theft / Synthetic Identity** | Using another person's identity, or a fabricated composite identity, to originate a loan |
| **Collusion** | Two or more parties in the origination chain (broker, appraiser, title agent, attorney) coordinating to commit fraud |
| **SAR** | Suspicious Activity Report — a mandatory filing with FinCEN (the US Treasury's financial intelligence unit) when a financial institution suspects fraud or money laundering |
| **FinCEN** | Financial Crimes Enforcement Network — the US Treasury bureau that receives and analyzes SAR filings |
| **Graph Pattern** | A recurring structural configuration in a network (e.g., multiple borrowers all connected to the same broker and the same appraiser) that signals a possible fraud ring |

### Pain Points

- No cross-deal fraud visibility — the most damaging fraud operates at the network level, which only becomes visible when data from multiple deals is combined
- Reactive detection — fraud is found through loss investigation, not through origination-stage screening
- No systematic broker performance tracking — brokers who consistently originate high-default loans are not systematically identified and excluded from future deals
- SAR filings are manual and inconsistent — the firm has no systematic process for identifying SAR-reportable activity across its loan portfolio

### Our Proposal

We introduce a **Persistent Fraud Intelligence Network** built on the Knowledge Graph.

The fundamental business change is the shift from single-deal fraud screening to **cross-deal, cross-vintage pattern detection**. Because the Knowledge Graph retains every broker, appraiser, borrower, and property across all deals the Firm has ever processed, patterns that are invisible at the deal level become visible at the network level.

The system maintains a continuously updated **Broker Risk Score** — a composite measure incorporating each broker's historical delinquency rate, their presence in previously flagged fraud investigations, their network connections to flagged appraisers, and statistical anomalies in the loans they originate. A broker's score is updated every month as new servicer performance data arrives.

When a new tape arrives, every broker in the pool is immediately looked up against this intelligence network. A broker with a score above a defined threshold triggers automatic escalation to enhanced due diligence before any human analyst has opened a file.

All fraud-related decisions follow a strict **Human-in-the-Loop** protocol: the system flags and evidences; a human investigator makes the final determination before any broker is blacklisted or any SAR is filed. The evidence chain supporting every decision is permanently recorded in the Knowledge Graph.

---

## Phase 9: Macro & Geopolitical Intelligence Layer (Cross-Cutting)

### Current State

Macro and geopolitical factors — interest rate movements, regional housing market trends, employment conditions, natural disasters, regulatory changes — profoundly affect mortgage pool performance. Currently, macro awareness in the monitoring and structuring process depends entirely on individual analyst judgment. There is no systematic process for incorporating macroeconomic context into pool risk assessment.

Structuring analysts adjust their stress scenarios informally based on current market conditions. Portfolio managers update their views on pools based on news they happen to read. None of this is systematic, documented, or connected to the loan-level data in any structured way.

### Key Terms

| Term | Definition |
|------|-----------|
| **HPA** | Home Price Appreciation — the rate at which property values are rising or falling in a given geography. The single most important macro variable for MBS performance |
| **MSA** | Metropolitan Statistical Area — a geographic unit used to measure housing market conditions at the city/metro level |
| **FHFA HPI** | Federal Housing Finance Agency House Price Index — the primary government measure of US home price changes |
| **FFR** | Federal Funds Rate — the US central bank's benchmark overnight interest rate, set by the Federal Open Market Committee (FOMC) |
| **FOMC** | Federal Open Market Committee — the Federal Reserve committee that sets US monetary policy. Its decisions on interest rates directly affect mortgage rates and refinancing behavior |
| **Basis Points (bps)** | A unit of measurement for interest rates and spreads: 1 basis point = 0.01%. A rate change from 5.00% to 5.25% is a 25 basis point change |
| **Spread Widening / Tightening** | When MBS spreads widen, investor required yields increase (prices fall); when spreads tighten, required yields decrease (prices rise). Driven by risk appetite and supply/demand |
| **Geopolitical Risk** | Events (wars, sanctions, trade policy changes, elections) that affect economic conditions and, indirectly, housing markets and borrower financial health |
| **Natural Disaster Exposure** | The concentration of a pool's properties in geographies subject to hurricane, flood, wildfire, or earthquake risk — an often-overlooked risk factor |
| **FEMA Flood Zone** | FEMA's classification of a property's flood risk — properties in high-risk zones require flood insurance and face elevated loss severity in flood events |
| **Regulatory Change Risk** | Changes to servicing guidelines, foreclosure laws, eviction moratoriums, or lending standards that affect pool performance or servicer behavior |

### Pain Points

- Macro context is applied informally and inconsistently across analysts and deals
- Geographic risk (natural disaster, regional economic concentration) is assessed at origination but not monitored dynamically as conditions evolve
- Regulatory changes (eviction moratoriums, forbearance programs) that affect pool performance are not systematically incorporated into monitoring
- No mechanism to connect a macro event (e.g., a major employer announcing layoffs in a specific city) to the pools with high exposure to that geography

### Our Proposal

We introduce a **Monthly Macro Intelligence Cycle** operated by AI agents.

On the first business day of each month, after servicer data is ingested, a set of AI agents automatically collect current macro data across a defined set of indicators: the Federal Reserve's latest rate decisions and forward guidance, HPA by MSA, unemployment by state, MBS spread levels, FEMA disaster declarations, and significant regulatory announcements. These data points are structured, compared to the prior month, and stored as a macro environment node in the Knowledge Graph.

The system then automatically calculates each pool's **macro exposure profile** — which macro factors are most relevant to its geographic and borrower composition — and adjusts pool risk scores accordingly. A pool heavily concentrated in a metro that just saw a 3% single-month home price decline will see its risk score updated before the delinquency data catches up.

This creates a **leading indicator** capability that the current process entirely lacks: the ability to anticipate stress before it appears in payment behavior, enabling proactive rather than reactive portfolio management.

---

## Summary: Where We Intervene and Why It Matters

| Phase | Current State | Our Intervention | Business Impact |
|-------|--------------|-----------------|----------------|
| Loan Tape Acquisition | Manual receipt, informal QC, no institutional memory | Automated ingestion, originator performance history from KG | Faster triage, consistent quality, retained institutional knowledge |
| Due Diligence | Sample-based, manual document review, single-deal fraud screening | AI Document Reader (HITL), 100% loan statistical screening, cross-deal fraud pattern detection | Reduced third-party costs, higher defect capture rate, fraud visible before losses |
| Pool Optimization | Excel-based, manual scenario testing, analyst-dependent | Graph-based constrained optimization, systematic scenario library | Better pools, faster structuring, learning from historical performance |
| Rating Agency Process | Manual submissions, iterative rounds, no pre-submission simulation | Pre-submission simulation, automated disclosure consistency check | Fewer rating agency rounds, lower error risk in disclosure documents |
| Investor Distribution | Mandate knowledge in salespeople's heads, no systematic targeting | Mandate-matched call list generation, automated suitability pre-check | Higher book-building efficiency, reduced regulatory exposure, institutional knowledge retained |
| Deal Closing | Manual data integrity between structuring and legal docs | Automated consistency check between KG data and closing documentation | Reduced operational errors, cleaner audit trail |
| Post-Securitization Monitoring | Reactive, backward-looking, individual pool reviewed in isolation | Proactive alerts, cross-pool pattern recognition | Problems caught earlier, fewer surprise losses |
| Fraud Detection | Rules-based, single-deal, reactive (post-loss) | Cross-deal network pattern detection, continuously updated broker risk scores | Fraud caught before deal entry, systematic SAR identification |
| Macro Intelligence | Informal, analyst-dependent, not connected to loan data | Monthly AI agent cycle, pool-level macro exposure scoring | Leading indicator capability, proactive risk management |

---

## Glossary of Acronyms

| Acronym | Full Form |
|---------|----------|
| ABS | Asset-Backed Security |
| AVM | Automated Valuation Model |
| bps | Basis Points |
| CDR | Constant Default Rate |
| CE | Credit Enhancement |
| CLTV | Combined Loan-to-Value |
| CPR | Constant Prepayment Rate |
| CUSIP | Committee on Uniform Securities Identification Procedures |
| DTI | Debt-to-Income |
| DTC | Depository Trust Company |
| ERISA | Employee Retirement Income Security Act |
| FEMA | Federal Emergency Management Agency |
| FHFA | Federal Housing Finance Agency |
| FFR | Federal Funds Rate |
| FinCEN | Financial Crimes Enforcement Network |
| FOMC | Federal Open Market Committee |
| HPA | Home Price Appreciation |
| HPI | House Price Index |
| IC | Interest Coverage |
| KG | Knowledge Graph |
| LTV | Loan-to-Value |
| MBS | Mortgage-Backed Security |
| MISMO | Mortgage Industry Standards Maintenance Organization |
| MSA | Metropolitan Statistical Area |
| NAIC | National Association of Insurance Commissioners |
| OC | Overcollateralization |
| OM | Offering Memorandum |
| PSA | Pooling and Servicing Agreement |
| R&W | Representations and Warranties |
| RBC | Risk-Based Capital |
| REO | Real Estate Owned |
| REMIC | Real Estate Mortgage Investment Conduit |
| SAR | Suspicious Activity Report |
| SOFR | Secured Overnight Financing Rate |
| TPDD | Third-Party Due Diligence |
| UPB | Unpaid Principal Balance |
| WAC | Weighted Average Coupon |
| WAL | Weighted Average Life |
| WALTV | Weighted Average LTV |
| WAM | Weighted Average Maturity |

---

*MBS Intelligence Platform — Business Planning Document v2.0*
*Architecture design to follow as a separate document*
