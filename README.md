# CVD Population Health Impact Simulator

> **Demo prototype** — illustrative and discussion purposes only. Not intended for clinical or commercial decision-making.

A single-page interactive simulator for communicating the population-level impact of a CVD intervention to non-technical stakeholders. Designed to be run in a browser during presentations — no specialist knowledge required to operate it.

## How to open

Download `index.html` and open it in any modern browser (Chrome, Firefox, Edge, Safari). No installation, no server, no internet connection required.

## What it does

Set five population parameters and one intervention slider, then instantly see how many deaths could be avoided and how many life-years could be preserved:

| Input | Description |
|---|---|
| Country | Sweden or Finland (determines life table and CVD incidence data) |
| Population size | Number of people in the cohort |
| Age band | Starting age in 5-year increments (30–34 through 80–84) |
| % Female | Sex mix of the cohort |
| Time horizon | 1, 5, or 10 years |
| CVD mortality reduction | Intervention effect as a relative % reduction (0–60%) |

**Outputs:**
- CVD events in the baseline arm
- Deaths avoided vs. baseline
- Life-years preserved (population level)
- Individual survival gain (% points)
- Survivors over time chart
- Cumulative CVD incidence chart

## Demo limitations

This version uses embedded Swedish and Finnish life table data (2022) and CVD incidence rates as illustrative hardcoded values. It is a prototype to demonstrate the model concept and output narrative — not a validated tool for real population analysis.

## Technical notes

- Self-contained static HTML file — no build step
- Plotly.js loaded from CDN (internet connection required for charts)
- All simulation logic runs in the browser via inline JavaScript
- Life table data for Sweden and Finland (ages 0–110) embedded directly in the HTML

## See also

- [`../heor-calculator/`](../heor-calculator/) — a more detailed HEOR calculator with cost-effectiveness outputs (cost per QALY, net monetary benefit), designed for health economist audiences
