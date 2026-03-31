# Home Buying Pipeline — Project Context

## What This Project Is

An automated multi-agent pipeline that guides a home buyer through every stage of the purchase
process — from assessing financial readiness through preparing for closing day. Each agent
plays a specific professional role (financial advisor, mortgage specialist, neighborhood researcher,
buyer's agent, inspection coordinator, closing specialist) and hands off structured data to the next.

The pipeline is built for Sarah & James Mitchell buying in the Austin, TX metro area, but is
fully configurable via `config/buyer-profile.yaml` and `config/search-criteria.yaml`.

---

## Data Model

Every agent reads and writes structured data files. This table is the authoritative reference.

| File | What It Contains | Who Writes | Who Reads |
|---|---|---|---|
| `config/buyer-profile.yaml` | Buyer demographics, income, debts, savings, deal-breakers | Human | All agents |
| `config/search-criteria.yaml` | Price range, bedrooms, location, scoring weights | Human + financial-advisor | neighborhood-researcher, buyer-agent |
| `data/financial-assessment.json` | DTI, affordable price range, savings check, credit tier | financial-advisor | mortgage-specialist, buyer-agent, closing-specialist |
| `data/mortgage-package.json` | Loan type comparison, rate estimates, PMI, docs checklist | mortgage-specialist | buyer-agent, closing-specialist |
| `data/neighborhood-reports/<name>.json` | Per-neighborhood: schools, crime, commute, taxes, market data | neighborhood-researcher | buyer-agent |
| `data/neighborhood-comparison.json` | Side-by-side comparison matrix, ranked recommendations | neighborhood-researcher | buyer-agent, daily-listing-alert |
| `data/listing-analysis.json` | Scored listings, comp analysis, recommended listing | buyer-agent | buyer-agent (draft-offer) |
| `data/offer-terms.json` | Complete offer: price, EMD, contingencies, dates, escalation | buyer-agent | review-offer, closing-specialist |
| `data/inspection-checklist.json` | What the inspector should check (pre-inspection brief) | inspection-coordinator | inspector (external) |
| `data/inspection-report-raw.md` | Raw inspection report text (provided by buyer post-inspection) | Human | inspection-coordinator |
| `data/inspection-report.json` | Categorized findings: deal-breaker / negotiate / cosmetic | inspection-coordinator | buyer-agent, closing-specialist |
| `data/repair-estimates.json` | Cost estimates per negotiate item, priority order | inspection-coordinator | buyer-agent |
| `data/repair-negotiation.json` | Repair request terms, seller response tracking | buyer-agent | closing-specialist |
| `data/closing-checklist.json` | All closing tasks with status, deadlines, responsible party | closing-specialist | All |
| `data/closing-costs.json` | Itemized closing cost breakdown, cash-to-close calculation | closing-specialist | buyer-agent |
| `data/new-listings-alert.json` | Daily scan results — new matches since yesterday | neighborhood-researcher | Human |
| `reports/financial-readiness.md` | Human-readable financial health report | financial-advisor | Buyer |
| `reports/mortgage-guide.md` | Loan options, rate estimates, docs checklist | mortgage-specialist | Buyer |
| `reports/neighborhood-guide.md` | Neighborhood comparison with ranked recommendations | neighborhood-researcher | Buyer |
| `reports/offer-summary.md` | Full offer terms for buyer review before submission | buyer-agent | Buyer (manual gate) |
| `reports/inspection-summary.md` | Inspection findings with recommendation | inspection-coordinator | Buyer |
| `reports/repair-request.md` | Professional repair request letter to seller | buyer-agent | Buyer, Listing Agent |
| `reports/closing-guide.md` | Complete closing roadmap: timeline, costs, docs, move-in | closing-specialist | Buyer |
| `reports/purchase-summary.md` | Final transaction summary and congratulations | closing-specialist | Buyer |

---

## Domain Terminology

**Financial:**
- **DTI (Debt-to-Income Ratio)**: Monthly debt payments ÷ gross monthly income. Front-end DTI uses only housing costs; back-end uses all debts. Most lenders want front-end ≤28%, back-end ≤43%.
- **LTV (Loan-to-Value Ratio)**: Loan amount ÷ property value. Below 80% LTV = no PMI required on conventional loans.
- **PMI (Private Mortgage Insurance)**: Required on conventional loans with <20% down. Costs 0.5-1.5% of loan annually. Cancels when LTV reaches 80%.
- **Pre-approval**: Lender has verified income, assets, and credit and commits to lending up to a specific amount. Different from pre-qualification (no verification).

**Offer & Negotiation:**
- **Earnest Money Deposit (EMD)**: Good-faith deposit held in escrow. Typically 1-3% of offer price. Forfeited if buyer backs out without contingency protection.
- **Contingency**: A condition that must be met for the sale to proceed. Common: financing, inspection, appraisal. Removing contingencies strengthens offers but increases buyer risk.
- **Escalation Clause**: Offer automatically increases by X dollars above any competing offer, up to a maximum price.
- **Comparable Sales (Comps)**: Recent sold prices for similar properties nearby. Used to determine fair market value.
- **Days on Market (DOM)**: How long a listing has been active. Low DOM = competitive market; high DOM = negotiating leverage.

**Inspection:**
- **Due Diligence Period**: Time after accepted offer when buyer can inspect, investigate, and back out. Typically 7-10 days.
- **Deal-breaker Defects**: Structural failure, major foundation issues, active mold, major water intrusion, buried oil tanks — issues so serious the buyer should walk away.
- **Negotiate Items**: Significant defects that don't make the home unsafe or unlivable but warrant repair credits or seller action.
- **Seller Credit**: Seller reduces the purchase price or pays buyer's closing costs in lieu of making repairs. Generally preferred over asking seller to repair.

**Closing:**
- **Title Search**: Review of public records to verify the seller owns the property free and clear of liens or other encumbrances.
- **Title Insurance**: Protects against future claims against the title. Lender's policy is required; owner's policy is optional but strongly recommended.
- **Escrow**: A neutral third party holds funds and documents during the transaction and disburses them at closing.
- **Closing Disclosure (CD)**: Final loan document showing exact closing costs and loan terms. Must be received 3 business days before closing.
- **Cash to Close**: Total funds the buyer must bring to closing = down payment + closing costs - any credits.
- **Appraisal**: Independent assessment of the property's fair market value ordered by the lender. If appraised value < purchase price, buyer must cover the gap or renegotiate.

---

## Key Invariants

These rules must never be violated by any agent:

1. **Offer price must not exceed the pre-approved maximum** from `data/financial-assessment.json`. This is a hard ceiling — escalation clause maximums must also respect this limit.

2. **Deal-breakers are non-negotiable.** If `inspection-coordinator` identifies a deal-breaker, the verdict MUST be `walk-away`. An agent may not recommend "proceed" or "negotiate" when a deal-breaker is present.

3. **Financial calculations must be consistent across agents.** The mortgage package must use the same price ranges and down payment amounts as the financial assessment. The closing cost breakdown must use the actual accepted offer price from offer-terms.json.

4. **Contingency periods are contractual deadlines.** If the inspection period is 7 days, the inspection and response must be completed within 7 calendar days of acceptance. The closing timeline must reflect actual contract dates.

5. **Listings that fail must-haves or trigger deal-breakers are rejected.** The scoring system is a guide, but hard failures cannot be overridden by high scores in other categories.

---

## Running This Example

```bash
cd examples/home-buyer

# Edit buyer profile and search criteria first
nano config/buyer-profile.yaml
nano config/search-criteria.yaml

# Start the daemon and run the full pipeline
ao daemon start
ao queue enqueue --title "home-buyer" --description "Run home buying pipeline for Mitchell family" --workflow-ref home-buying-pipeline

# Watch progress
ao daemon stream --pretty
```

### Manual Gate: Offer Review

The pipeline will pause at `review-offer` and wait for buyer input. Check `reports/offer-summary.md`
and then resume with your decision:

```bash
ao task list   # Find the paused task ID
ao task get <task-id>   # See the manual gate prompt
```
