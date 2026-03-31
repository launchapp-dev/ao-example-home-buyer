# Home Buying Pipeline — Build Plan

## Overview

An automated home buying pipeline that guides a buyer through the entire purchase process: financial readiness assessment, mortgage pre-approval preparation, neighborhood research, house hunting criteria matching, offer strategy and negotiation, inspection coordination, and closing preparation. Uses web research (fetch MCP + Playwright) for real neighborhood data, school ratings, and listing research.

## Agents

| Agent | Model | Role |
|---|---|---|
| **financial-advisor** | claude-sonnet-4-6 | Assesses buyer financial readiness — income, debt, savings, credit score analysis. Calculates DTI ratios, affordable price range, down payment scenarios, and monthly payment estimates. |
| **mortgage-specialist** | claude-haiku-4-5 | Prepares mortgage pre-approval package — compares loan types (conventional, FHA, VA, USDA), calculates rates and PMI, identifies required documents, generates lender comparison matrix. |
| **neighborhood-researcher** | claude-sonnet-4-6 | Researches neighborhoods using web fetch — school ratings, crime data, commute times, walkability, property tax rates, HOA info, appreciation trends. Builds comparison matrix. |
| **buyer-agent** | claude-sonnet-4-6 | Evaluates listings against buyer criteria, formulates offer strategy (aggressive/market/conservative), drafts offer terms, handles counter-offer analysis. Decision-making agent for offers. |
| **inspection-coordinator** | claude-haiku-4-5 | Coordinates home inspection — generates inspection checklist, analyzes inspection report findings, categorizes issues (deal-breaker/negotiate/cosmetic), calculates repair cost estimates. |
| **closing-specialist** | claude-sonnet-4-6 | Manages closing preparation — title search checklist, closing cost breakdown, document checklist, timeline coordination, final walkthrough prep, move-in checklist and utility setup. |

## Phase Pipeline

```
1. assess-finances          → Financial readiness report (income, debt, savings, credit)
2. prepare-mortgage         → Mortgage pre-approval package (loan comparison, docs needed)
3. research-neighborhoods   → Neighborhood comparison matrix (schools, safety, commute)
4. evaluate-listings        → Listing analysis + offer strategy recommendation
   ├── on "aggressive"    → draft-offer
   ├── on "market"        → draft-offer
   ├── on "conservative"  → draft-offer
   └── on "pass"          → research-neighborhoods (rework — search more areas)
5. draft-offer              → Offer terms document
6. review-offer             → Decision gate: submit / revise / withdraw
   ├── on "submit"        → coordinate-inspection
   ├── on "revise"        → draft-offer (rework)
   └── on "withdraw"      → research-neighborhoods (start over)
7. coordinate-inspection    → Inspection report analysis + repair estimates
   ├── on "proceed"       → prepare-closing
   ├── on "negotiate"     → negotiate-repairs (rework to draft counter)
   └── on "walk-away"     → research-neighborhoods (start over)
8. negotiate-repairs        → Repair negotiation terms
9. prepare-closing          → Closing checklist, cost breakdown, timeline
10. generate-closing-report → Final summary report + move-in checklist
```

## Workflow Routing

- **evaluate-listings** is the first major decision point — pass sends buyer back to research more neighborhoods
- **review-offer** is a manual approval gate — buyer must approve the offer before it's submitted
- **coordinate-inspection** determines whether to proceed, negotiate repairs, or walk away
- **negotiate-repairs** feeds back into inspection coordination for re-evaluation
- max_rework_attempts: 3 on evaluate-listings (don't search forever)
- max_rework_attempts: 2 on review-offer (don't revise endlessly)
- max_rework_attempts: 2 on coordinate-inspection (negotiate once or twice, then decide)

## MCP Servers

| Server | Purpose |
|---|---|
| `filesystem` | Read/write all data files, reports, and checklists |
| `sequential-thinking` | Structured reasoning for financial calculations, offer strategy, inspection analysis |
| `fetch` | Research neighborhood data, school ratings, property tax info from web |

## Data Model — Files

| File | What It Contains | Who Reads | Who Writes |
|---|---|---|---|
| `config/buyer-profile.yaml` | Buyer demographics, income, savings, credit score, preferences, must-haves, deal-breakers | All agents | Human (provided) |
| `config/search-criteria.yaml` | Location preferences, price range, bedrooms, sq ft, lot size, school district requirements | neighborhood-researcher, buyer-agent | financial-advisor (sets price range), human (preferences) |
| `data/financial-assessment.json` | DTI ratios, affordable price range, down payment scenarios, monthly payment estimates | mortgage-specialist, buyer-agent, closing-specialist | financial-advisor |
| `data/mortgage-package.json` | Loan type comparison, rate estimates, PMI calculations, required documents checklist | buyer-agent, closing-specialist | mortgage-specialist |
| `data/neighborhood-reports/` | Per-neighborhood research: schools, crime, commute, taxes, walkability, appreciation | buyer-agent | neighborhood-researcher |
| `data/neighborhood-comparison.json` | Side-by-side comparison matrix of all researched neighborhoods | buyer-agent | neighborhood-researcher |
| `data/listing-analysis.json` | Evaluated listings with scores against criteria, comparable sales, days on market | buyer-agent | buyer-agent |
| `data/offer-terms.json` | Offer price, earnest money, contingencies, closing date, escalation clause, terms | review-offer phase, closing-specialist | buyer-agent |
| `data/inspection-report.json` | Inspection findings categorized: structural, electrical, plumbing, HVAC, roof, cosmetic | inspection-coordinator, buyer-agent | inspection-coordinator |
| `data/repair-estimates.json` | Cost estimates for each inspection finding, priority ranking | buyer-agent, closing-specialist | inspection-coordinator |
| `data/repair-negotiation.json` | Repair request terms, seller response, credits vs fixes | closing-specialist | buyer-agent |
| `data/closing-checklist.json` | All closing tasks with status, deadlines, responsible parties | All agents | closing-specialist |
| `data/closing-costs.json` | Itemized closing cost breakdown: lender fees, title, insurance, taxes, prepaid | buyer-agent | closing-specialist |
| `reports/financial-readiness.md` | Human-readable financial readiness report | Buyer | financial-advisor |
| `reports/neighborhood-guide.md` | Formatted neighborhood comparison guide | Buyer | neighborhood-researcher |
| `reports/offer-summary.md` | Offer terms summary for buyer review | Buyer | buyer-agent |
| `reports/inspection-summary.md` | Inspection findings summary with recommendations | Buyer | inspection-coordinator |
| `reports/closing-guide.md` | Complete closing guide: timeline, costs, documents needed, move-in checklist | Buyer | closing-specialist |

## Sample Data

Create realistic sample data for a buyer persona:

**Buyer: Sarah & James Mitchell**
- Combined household income: $165,000/year
- Savings: $72,000 (down payment + reserves)
- Credit scores: 748 (Sarah), 721 (James)
- Current rent: $2,400/month
- Student loans: $34,000 remaining
- Car payment: $450/month
- Looking in: Austin, TX metro area
- Budget: ~$400K-$500K
- Need: 3+ bedrooms, 2+ baths, good schools, <30 min commute to downtown
- Nice-to-have: garage, yard, updated kitchen
- Deal-breakers: flood zone, foundation issues, HOA >$300/month

## README Outline

1. **What This Is** — Automated home buying pipeline powered by AO
2. **The Pipeline** — Visual flow diagram of all phases with decision points
3. **Agents** — Table of agents, models, and roles
4. **How It Works** — Step-by-step walkthrough of a typical run
5. **Configuration** — How to customize buyer-profile.yaml and search-criteria.yaml
6. **Sample Run** — What the output looks like for the Mitchell family scenario
7. **Data Files** — Complete data model reference
8. **Decision Points** — Explanation of offer strategy, inspection, and closing decisions
9. **Running It** — `ao daemon start` instructions

## CLAUDE.md Outline

1. **What This Project Is** — One paragraph summary
2. **Data Model** — Table of every file, who reads/writes it
3. **Domain Terminology** — DTI, LTV, PMI, earnest money, contingencies, escrow, title insurance, appraisal, comparable sales, days on market, escalation clause, due diligence period
4. **Key Invariants** — Financial calculations must be consistent, offer must not exceed pre-approved amount, inspection deal-breakers are non-negotiable, closing timeline must account for all contingency periods

## Directory Structure

```
examples/home-buyer/
├── .ao/workflows/
│   ├── agents.yaml
│   ├── phases.yaml
│   ├── workflows.yaml
│   └── mcp-servers.yaml
├── config/
│   ├── buyer-profile.yaml
│   └── search-criteria.yaml
├── data/
│   ├── neighborhood-reports/     (created by neighborhood-researcher)
│   └── (all .json files created by agents)
├── reports/                      (human-readable reports)
├── CLAUDE.md
└── README.md
```
