# Feature Platform — Agentic Recovery Workflow

When a governance feature breaks, the platform should not just say *what* broke. It should
diagnose why, find what can replace it, and hand a reviewable change proposal to the strategy
platform — with a human confirming every write.

**Live prototype (v0.4_1):** https://yg674-dev.github.io/feature-recommendation-demo/
· bilingual, toggle in the top-right, no build step
**Full PRD:** [`docs/agentic-workflow-prd.md`](docs/agentic-workflow-prd.md) — this README is the summary.

---

## 1. Business goal

Governance strategies run on features, so a broken feature is a broken policy — silently, until
someone notices. The goal is to shorten the interval between a feature going anomalous and a
strategy change landing safely, **without loosening the review that keeps bad changes out of
production**.

That means three things the business can measure: anomaly localization time against its current
baseline; the share of replacement decisions that arrive reviewable rather than merely
recommended; and the number of strategy changes reaching production without human confirmation,
which must stay at zero.

The gap this closes: a feature registry plus RAG Q&A gets a user as far as *finding and
understanding* features. In the IDSP strategy production lifecycle that is not the job. When a
feature goes anomalous, the user needs to know why, which strategies are affected, whether to
replace or fall back, and how to get that change into IDSP safely — which is the loop this
builds: **anomaly → feature card → replacement candidates → IDSP strategy change proposal**, with
the agent proposing and a human confirming.

## 2. Problem

- **Discovery ≠ recovery.** Registry and Q&A answer "what is this feature," not "this feature is
  broken at 3am, what do I do."
- **The feature card is static.** It explains a feature but carries no current health, no strategy
  usage, no trace/metrics evidence, and no candidate comparison — so it cannot support a recovery
  decision.
- **A bare recommendation invites misuse.** If Ask AI returns "recommended features" and nothing
  else, users adopt a candidate whose behavior differs in ways nobody checked.
- **The integration boundary is undefined.** Without an explicit read → propose → human-confirmed-write
  model, an agent can end up editing strategies directly or routing around review.

## 3. Users and jobs to be done

| User | Job |
| --- | --- |
| **Strategy owner** during an incident | "Tell me what broke, what it affects, and what I can safely swap in — with the difference analysis, not just a name." |
| **Platform on-call / DA** | "Cut how long it takes to localize an anomaly across data quality, feature service, strategy consumption, and business impact." |
| **IDSP owner** | "Give me a change proposal complete enough to review, and never let anything reach production without my confirmation." |

## 4. Goals and non-goals

**Goals, by priority**

| | Goal | Coverage |
| --- | --- | --- |
| **P0** | Minimum loop: anomaly → feature card → replacement candidates | 4 monitoring metric categories (data quality, feature service, strategy consumption, business impact) · 3 card modes (Discovery, Diagnosis, Replacement) · 5 candidate fit levels (full substitute, partial substitute, fallback only, derived candidate, not suitable) · **100% of replacement analyses carry fit label, difference analysis, risk, and validation plan** |
| **P1** | IDSP strategy change proposal | 100% of proposals carry impacted strategy, current and candidate feature, replacement type, condition change, expected impact, risk, validation plan, rollout plan, rollback plan · **100% human-confirmed** |
| **P2** | Lineage and derived features | 8 lineage relations — `derived_from`, `same_source_as`, `sibling_of`, `used_by_strategy`, `replaces`, `fallback_for`, `conflicts_with`, `deprecated_by`. Where nothing substitutes, generate a derived-feature proposal into new-feature entry + agent evaluation |
| **P3** | Agentic recovery loop | Agent detects, diagnoses, finds candidates, proposes; monitors after rollout and suggests rollback on failure — **still zero auto-bypass of human confirmation** |

**Non-goals** — the agent does not modify strategies directly, does not launch a change, and does
not make the adoption decision. It produces a reviewable artifact.

## 5. Solution

**Pages**

1. **Feature monitoring dashboard** — anomalies across the four metric categories.
2. **Feature drilldown** — health, usage, and the trace/metrics evidence behind an anomaly.
3. **Agent diagnosis panel** — why this feature went anomalous, and what it affects.
4. **Enhanced feature card** — Discovery / Diagnosis / Replacement modes, with a fit label on
   every replacement candidate.
5. **Lineage & derived-feature explorer** — the 8 relations above, and the path to a derived
   proposal when nothing substitutes.

**Behind the pages**

Replacement discovery · agent skill split · IDSP integration (read → proposal → human-confirmed
write) · a state machine for the recovery lifecycle · and a strategy-change-proposal contract that
defines what a proposal must contain before a human sees it.

## 6. Functional requirements

1. **No recommendation ships bare.** Every replacement candidate carries a fit label, difference
   analysis, risk, and validation plan — the analysis is the deliverable, the name is not.
2. **The integration is layered and one-directional.** Read integration, then proposal integration,
   then human-confirmed write handoff. The agent never writes to a strategy.
3. **Every failure mode is named, not swallowed.** The guardrail framework defines explicit cases —
   `LOW_CONFIDENCE_REPLACEMENT`, `SOURCE_SHARED_RISK` (a candidate sharing the broken source is not
   a substitute), `IDSP_USAGE_CONTEXT_MISSING`, `VALIDATION_PLAN_MISSING` — each with its own
   interaction.
4. **A proposal is complete or it is not a proposal.** The contract's required fields gate handoff.
5. **Rollback is designed in**, not added after: every proposal carries a rollout and a rollback plan.

## 7. Success metrics

- **North star** — time from anomaly detection to a confirmed strategy change.
- **Leading** — share of anomalies with an agent diagnosis · share of replacement analyses complete
  on all four required parts · derived-proposal evaluation pass rate.
- **Lagging** — anomaly localization time against baseline · proposals accepted without rework.
- **Guardrail** — `LOW_CONFIDENCE_REPLACEMENT` and `SOURCE_SHARED_RISK` rates, and **zero**
  strategy changes reaching production without human confirmation.

## 8. What's in this repo

| Path | What it is |
| --- | --- |
| `index.html` | The current prototype, v0.4_1 |
| [`docs/agentic-workflow-prd.md`](docs/agentic-workflow-prd.md) | The full PRD — bilingual, with the feature list, user-story routes, page specs, backend logic, and the guardrail framework |
| `prototypes/` | Earlier iterations — [v0.4](https://yg674-dev.github.io/feature-recommendation-demo/prototypes/v0.4.html) · [v0.2](https://yg674-dev.github.io/feature-recommendation-demo/prototypes/v0.2.html) · [v0.1](https://yg674-dev.github.io/feature-recommendation-demo/prototypes/v0.1.html) |
| `flowchart/` | Policy-issue flowchart — [live](https://yg674-dev.github.io/feature-recommendation-demo/flowchart/agentic-policy-issue-flowchart.html) · [PNG](flowchart/agentic-policy-issue-flowchart.png) |

The prototype versions are kept deliberately: v0.1 through v0.4_1 show how the card model and the
proposal contract tightened across reviews.

## Running locally

```bash
git clone https://github.com/yg674-dev/feature-recommendation-demo.git
cd feature-recommendation-demo
open index.html
```

Self-contained HTML — no build step, no server.
