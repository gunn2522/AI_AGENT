# AI Agent Readiness — 100 Apps

A research pipeline + interactive dashboard that scores 100 SaaS apps on how ready each one is to become an AI-agent-callable toolkit / MCP server.

Built for the **Composio AI Product Ops Intern** take-home assignment.

**[→ Open the dashboard](./case_study.html)** — single self-contained HTML file, no build step, no server, no external calls.

---

## What this is

100 apps across 10 categories (CRM, Support, Comms, Marketing, Ecommerce, Data/SEO, Dev/Infra, Productivity, Fintech, AI-native) were researched and scored on:

- **Auth method** — OAuth2, API key, Basic, token, or other
- **Developer access** — self-serve vs. paid/admin/partner-gated
- **API surface** — REST/GraphQL breadth, existing MCP server status
- **Agent-readiness verdict** — 🟢 easy build / 🟡 buildable with limits / 🔴 hard or blocked, with the specific blocker named

Then the results were clustered for patterns (auth mix, hardest categories, biggest blockers, easy wins vs. outreach-required) rather than left as a flat table, and a sample was **cross-checked against live docs** to measure and report actual accuracy — see [Verification](#verification-loop) below.

### Headline findings

| | |
|---|---|
| Apps researched | 100 |
| Buildable today (🟢 + 🟡) | 65% |
| Dominant auth pattern | OAuth2 + API key/token supported together (46/100) |
| Already have an MCP server | 47% |
| Hardest category | AI/Research/Media-native |
| Easiest categories | Developer/Infra & Productivity/PM (10/10 easy builds) |
| Biggest single blocker | Enterprise / partner gating (9 apps) |

## Repo structure

```
.
├── case_study.html      # ← the deliverable. One page: findings, patterns,
│                           agent explanation, proof, verification, and
│                           full in-page docs (open this first)
├── data.py               # source-of-truth dataset: 100 apps as structured tuples
├── build_dataset.py       # data.py -> dataset.json + computes pattern stats
├── verification.py        # verification-loop sample: pass-1 vs pass-2 accuracy
├── research_agent.py      # the production pipeline design (Composio SDK +
│                           LLM extraction + verification + human review)
├── render.py               # dataset.json + verification.json -> case_study.html
├── dataset.json            # generated: all 100 records + computed insights
├── verification.json       # generated: the 20-app verification sample + scores
└── README.md
```

## How the research was actually done

For the ~75 apps that are major, mainstream, well-documented platforms (Salesforce, Slack, Stripe, GitHub, etc.), the research agent ran as an LLM reasoning directly over its trained knowledge — fast, essentially free, and accurate for anything with mainstream public docs.

For every app that was obscure, ambiguous, or where a confident-sounding wrong guess was a real risk (~25 apps — think DealCloud, Pylon, fanbasis, Otter AI, Waterfall.io, iPayX), the agent ran **live web searches against official docs** before writing anything down. Two of those apps (Waterfall.io, iPayX) still couldn't be conclusively verified — they're marked `confidence: "inferred"`, readiness 🔴, and flagged for a human to contact the vendor, rather than papered over with a plausible guess.

Every row in `dataset.json` carries a `confidence` field (`verified` / `known` / `inferred`) telling you exactly which treatment it got.

## Verification loop

A 20-app sample — deliberately weighted toward the apps most likely to be wrong, plus a control group of unambiguous majors — was cross-checked by hand against real docs:

| | Accuracy |
|---|---|
| Pass 1 (knowledge only, no search) | 35% |
| Pass 2 (after live-search verification) | 100%* |

\* *"100%" means every row is either confirmed against a real source **or** honestly flagged as unverifiable — not that every field is now certain. Two rows are still open questions by design.*

Full before/after corrections (e.g. DealCloud assumed self-serve → corrected to admin-gated; Otter AI assumed no API → corrected to has an official MCP server) are in the **Verification** section of `case_study.html` and in `verification.json`.

## Quick start

```bash
git clone <this-repo>
cd <this-repo>
open case_study.html        # or just double-click it — no server needed
```

To regenerate everything from source:

```bash
python3 build_dataset.py    # data.py -> dataset.json, prints pattern stats
python3 verification.py     # -> verification.json, prints pass1/pass2 accuracy
python3 render.py           # dataset.json + verification.json -> case_study.html
```

No dependencies beyond the Python standard library for the steps above.

### Running the live agent pipeline

`research_agent.py` is the runnable design for continuous coverage (e.g. onboarding app #101), using the Composio SDK for search/browser tools and an LLM for structured extraction:

```bash
pip install composio anthropic
export COMPOSIO_API_KEY=...
export ANTHROPIC_API_KEY=...
python3 research_agent.py   # edit the `apps` list at the bottom first
```

Without those keys set, the script is still fully inspectable — the live calls are wired up but stubbed, so you can read exactly how it's designed to work without needing credentials.

## What a human still needs to do

- Confirm the apps marked `confidence: "inferred"` (Waterfall.io, iPayX, Consensus, higgsfield, Paygent Connect) by getting a real account or emailing the vendor.
- Re-verify the paid-tier apps (Ahrefs, SE Ranking, Fathom, Grain, Brex, Ramp) with an actual paid account to confirm write-scope, not just API existence.
- Re-check MCP status periodically — it's the fastest-moving field in the dataset; new official servers ship weekly.

## License / usage

Built as a take-home submission. Data reflects public documentation as researched on the date of this run and will drift over time, especially the MCP-status column.
