# Feature Recommendation Demo

Interactive HTML prototype for the Feature Platform Agentic Workflow — a
bilingual (中文 / English) walkthrough of feature discovery, anomaly diagnosis,
side-by-side comparison, replacement recommendation, and shadow-test simulation.

## Live prototype

**Current version (v0.4_1):** https://yg674-dev.github.io/feature-recommendation-demo/

Language toggle is in the top-right corner. Loads directly in any modern browser
— no build step, no server.

## Prototype iterations

| Version  | Live link                                                                                   | Local file                |
| -------- | ------------------------------------------------------------------------------------------- | ------------------------- |
| v0.4_1   | [open](https://yg674-dev.github.io/feature-recommendation-demo/)                            | `index.html`              |
| v0.4     | [open](https://yg674-dev.github.io/feature-recommendation-demo/prototypes/v0.4.html)        | `prototypes/v0.4.html`    |
| v0.2     | [open](https://yg674-dev.github.io/feature-recommendation-demo/prototypes/v0.2.html)        | `prototypes/v0.2.html`    |
| v0.1     | [open](https://yg674-dev.github.io/feature-recommendation-demo/prototypes/v0.1.html)        | `prototypes/v0.1.html`    |

## Supporting materials

- **PRD:** [`docs/agentic-workflow-prd.md`](docs/agentic-workflow-prd.md) — the
  corresponding product requirements document.
- **Policy-issue flowchart:** [live](https://yg674-dev.github.io/feature-recommendation-demo/flowchart/agentic-policy-issue-flowchart.html) · [`flowchart/agentic-policy-issue-flowchart.html`](flowchart/agentic-policy-issue-flowchart.html) · [PNG](flowchart/agentic-policy-issue-flowchart.png)

## Repository layout

```
feature-recommendation-demo/
├── index.html                    Main demo (v0.4_1)
├── docs/
│   └── agentic-workflow-prd.md   PRD
├── prototypes/
│   ├── v0.1.html                 Earlier iteration
│   ├── v0.2.html                 Earlier iteration
│   └── v0.4.html                 Earlier iteration
└── flowchart/
    ├── agentic-policy-issue-flowchart.html
    └── agentic-policy-issue-flowchart.png
```

## Running locally

```bash
git clone https://github.com/yg674-dev/feature-recommendation-demo.git
cd feature-recommendation-demo
open index.html
```
