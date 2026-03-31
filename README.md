# Home Buying Pipeline

An automated multi-agent pipeline that guides buyers through the entire home purchase journey — financial assessment, mortgage prep, neighborhood research, offer strategy, inspection coordination, and closing preparation.

---

## The Pipeline

```
                    ┌─────────────────────────────────────┐
                    │         Home Buying Pipeline         │
                    └─────────────────────────────────────┘

  ┌─────────────────┐    ┌─────────────────┐
  │ assess-finances │───▶│ prepare-mortgage │
  └─────────────────┘    └────────┬────────┘
   financial-advisor       mortgage-specialist
                                  │
                                  ▼
                    ┌─────────────────────────┐
               ┌───│  research-neighborhoods  │◀──────────────────────────────┐
               │   └─────────────────────────┘                                │
               │      neighborhood-researcher                                  │
               │              │                                                │
               │              ▼                                                │
               │   ┌─────────────────────┐                                    │
               │   │  evaluate-listings  │ ──[pass: no match]──────────────────┘
               │   └─────────────────────┘
               │        buyer-agent
               │              │ [aggressive / market / conservative]
               │              ▼
               │      ┌─────────────┐
               │      │ draft-offer │◀──── [revise]
               │      └──────┬──────┘             │
               │        buyer-agent                │
               │              │                   │
               │              ▼                   │
               │   ┌──────────────────┐           │
               │   │  review-offer    │ (MANUAL) ──┘
               │   │  [buyer decides] │
               │   └────────┬─────────┘
               │            │ [submit]
               │            │
               │            ▼
               │  ┌────────────────────────┐
               └──│ coordinate-inspection  │ ──[walk-away]─────────────────────┐
                  └────────────┬───────────┘                                   │
                   inspection-coordinator                                       │
                               │                                               │
                    ┌──────────┼─────────────┐                                 │
                [proceed]  [negotiate]   [walk-away]                           │
                    │          │               └─────────────────────────────── ┘
                    │     ┌────▼──────────┐
                    │     │negotiate-     │ ──[walk-away]──────────────────────┐
                    │     │  repairs      │                                     │
                    │     └──────┬────────┘                                    │
                    │       buyer-agent                                         │
                    │            │ [proceed]                                    │
                    └────────────┤                                              │
                                 ▼                                              │
                    ┌─────────────────────┐                                    │
                    │   prepare-closing   │◀───────────────────────────────────┘
                    └──────────┬──────────┘
                     closing-specialist
                               │
                               ▼
                    ┌──────────────────────────┐
                    │  generate-closing-report  │
                    └───────────────────────────┘
                         closing-specialist

  ┌─────────────────────────────────────────────────────────────────┐
  │ SCHEDULED: daily-listing-alert — runs weekdays at 8am           │
  │ Scans for new matching listings and writes alert to data/       │
  └─────────────────────────────────────────────────────────────────┘
```

---

## Agents

| Agent | Model | Role |
|---|---|---|
| **financial-advisor** | claude-sonnet-4-6 | Assesses DTI, affordable price range, savings adequacy, credit tier. Generates financial readiness report. |
| **mortgage-specialist** | claude-haiku-4-5 | Compares loan types (Conventional, FHA, VA, USDA), calculates PMI, builds required docs checklist. |
| **neighborhood-researcher** | claude-sonnet-4-6 | Fetches real neighborhood data — school ratings, crime, commute, market trends, property taxes. |
| **buyer-agent** | claude-sonnet-4-6 | Scores listings against criteria, recommends offer strategy, drafts complete offer terms. |
| **inspection-coordinator** | claude-haiku-4-5 | Categorizes inspection findings (deal-breaker/negotiate/cosmetic), estimates repair costs, recommends response. |
| **closing-specialist** | claude-sonnet-4-6 | Builds closing cost breakdown, timeline, required docs checklist, final walkthrough, move-in checklist. |

---

## Quick Start

```bash
cd examples/home-buyer

# 1. Customize your buyer profile
nano config/buyer-profile.yaml

# 2. Set your search criteria
nano config/search-criteria.yaml

# 3. Start the daemon
ao daemon start

# 4. Launch the pipeline
ao queue enqueue \
  --title "home-buyer" \
  --description "Run home buying pipeline" \
  --workflow-ref home-buying-pipeline

# 5. Watch it run
ao daemon stream --pretty

# 6. When pipeline pauses at review-offer, check the offer summary
cat reports/offer-summary.md

# 7. Resume with your decision (see task ID from ao task list)
ao task list
```

### Daily Listing Alerts (Optional)

The `listing-monitor` workflow runs on a weekday schedule to alert you to new matching listings:

```bash
# Enable scheduled alerts
ao daemon start --autonomous

# Manual run anytime
ao workflow run listing-monitor
```

---

## AO Features Demonstrated

| Feature | Where Used |
|---|---|
| **Multi-agent pipeline** | 6 specialized agents with distinct roles |
| **Decision contracts** | `evaluate-listings` (aggressive/market/conservative/pass), `coordinate-inspection` (proceed/negotiate/walk-away) |
| **Phase routing on verdict** | `pass` → back to neighborhood research; `walk-away` → restart search |
| **Manual approval gate** | `review-offer` pauses for buyer to approve before submitting |
| **Rework loops** | Offer revision (max 2 revisions), neighborhood retry (max 3 attempts) |
| **Scheduled workflows** | Daily 8am listing scan on weekdays |
| **Web fetch MCP** | Neighborhood researcher fetches real school ratings, crime data, market trends |
| **Sequential thinking MCP** | Financial calculations, offer strategy, inspection analysis |
| **Output contracts** | Structured JSON data files passed between agents |
| **Post-success merge** | Squash merge to main on pipeline completion |

---

## Requirements

### No API Keys Required
This pipeline uses only open MCP servers that require no authentication:
- `@modelcontextprotocol/server-filesystem` — file read/write
- `@modelcontextprotocol/server-fetch` — web research
- `@modelcontextprotocol/server-sequential-thinking` — structured reasoning

### Optional: Real Listing Data
By default, the `buyer-agent` fetches public listing data from Zillow, Redfin, or Realtor.com
via the fetch MCP. No API key needed — public pages only.

### Post-Inspection: Provide Your Inspection Report
After a real inspection, drop the report text into:
```
data/inspection-report-raw.md
```
The `coordinate-inspection` phase will analyze it. If the file is missing, the agent generates
a realistic sample report for demonstration.

---

## Output Files

After a full pipeline run:

```
reports/
├── financial-readiness.md   ← Your financial health assessment
├── mortgage-guide.md        ← Loan options and what you'll need
├── neighborhood-guide.md    ← Neighborhood comparison with recommendations
├── offer-summary.md         ← Complete offer terms (reviewed before submitting)
├── inspection-summary.md    ← Inspection findings and recommendation
├── repair-request.md        ← Seller repair/credit request letter (if needed)
├── closing-guide.md         ← Complete closing roadmap
└── purchase-summary.md      ← Final transaction summary
```

---

## Sample Buyer Scenario

**Sarah & James Mitchell — Austin, TX**

- Combined income: $165,000/year
- Savings: $72,000 (down payment + reserves)
- Credit scores: 748 / 721
- Current rent: $2,400/month
- Student loans: $34K remaining, $285/month
- Car payment: $450/month
- Looking for: 3+ bed, 2+ bath in Round Rock, Cedar Park, or Pflugerville
- Budget: $400K–$480K, targeting 10% down
- Must-haves: good schools (rating 7+), under 35 min commute to downtown Austin
- Deal-breakers: flood zone, foundation issues, HOA over $300/month

This profile is pre-loaded in `config/buyer-profile.yaml`. Run as-is to see a complete pipeline demo.
