# PRD: CVD Population Health Impact Simulator

## Problem Statement

Public health teams and HEOR analysts need a fast, interactive way to communicate the population-level impact of a CVD intervention to non-technical stakeholders. Current model outputs live inside R scripts and Jupyter notebooks that require specialist knowledge to run and interpret. There is no accessible interface that lets a user define a population, set a simple intervention effect, and immediately see how many deaths could be avoided and how many life-years could be preserved.

## Solution

A self-contained single-page HTML calculator — the **CVD Population Health Impact Simulator** — that takes five simple population inputs and one intervention input, then instantly displays both individual (Micro) and population (Macro) health outcomes across a chosen time horizon. It uses the project's existing Swedish and Finnish life tables and CVD incidence data as its embedded data source. No server, no dependencies beyond Plotly.

## User Stories

1. As a health economist, I want to select a reference country (Sweden or Finland), so that the model uses appropriate age- and sex-specific mortality rates.
2. As a health economist, I want to input a population size, so that outputs are expressed at a realistic population scale.
3. As a health economist, I want to select an age band in 5-year increments (e.g. 50–54), so that the simulation reflects a clinically relevant cohort age.
4. As a health economist, I want to set the percentage of females in the cohort, so that the model correctly blends male and female life table data.
5. As a health economist, I want to choose a time horizon of 1, 5, or 10 years, so that I can show short- and medium-term impact.
6. As a health economist, I want a single "CVD mortality reduction %" slider, so that I can model the intervention effect without needing to understand risk tiers.
7. As a health economist, I want to see expected CVD events in the baseline (no intervention) arm, so that I have a reference point.
8. As a health economist, I want to see deaths avoided with the intervention vs. baseline, so that I can communicate the mortality benefit.
9. As a health economist, I want to see the number of survivors over time as a chart, so that stakeholders can visually grasp the divergence between arms.
10. As a health economist, I want to see cumulative CVD incidence over time for both arms, so that I can show how the intervention shifts disease burden.
11. As a presentation user, I want a Micro KPI showing the improvement in individual survival probability, so that I can personalise the message to patients.
12. As a presentation user, I want a Macro KPI showing total life-years preserved at population level, so that I can communicate societal value to payers.
13. As a non-technical stakeholder, I want to reset all inputs to defaults with one click, so that I can start a new scenario quickly.
14. As a user on a laptop, I want the layout to be responsive, so that the calculator works in presentations and on smaller screens.
15. As a user, I want the visual style to match the existing HEOR calculator, so that both tools feel like a coherent product family.

## Implementation Decisions

### Modules

**1. Embedded Data Layer**
- Life table data for Sweden and Finland (male and female) is read from the project's `.txt` files and embedded as JavaScript arrays inside the HTML at build time (or hardcoded as pre-parsed constants). Data covers ages 0–110, mortality rate `mx` and `qx` per age per year.
- CVD incidence rates by age are derived from `SWE_m_short_idr.csv` (cause S002 = ischaemic heart disease / main CVD burden column) and embedded similarly. Finnish incidence uses the same rates scaled by the Finnish/Swedish mortality ratio for the chosen age band (pragmatic approximation given data availability).
- The data layer exposes two pure functions: `getMortalityRate(country, sex, age)` and `getCVDIncidenceRate(country, sex, age)`.

**2. Cohort Simulation Engine**
- Accepts: `{ country, popSize, ageBandMin, ageBandMax, pctFemale, horizon, mortReduction }`.
- Runs two arms: baseline and intervention (intervention arm applies `mortReduction` as a relative reduction to CVD-attributable mortality only).
- Uses an annual Markov cycle over `horizon` years. Cohort starts in a single "alive, CVD-free" state and transitions to "CVD event", "death (CVD)", or "death (other)" each cycle using blended male/female rates.
- Returns per-year arrays: `survivors`, `cvdEvents`, `cumulativeCVDIncidence`, and summary scalars: `deathsAvoided`, `lifeYearsPreserved`, `microSurvivalGain`.
- Pure function — no DOM access. Fully testable in isolation.

**3. UI / Render Layer**
- Left panel: country dropdown, population size number input, age band dropdown (5-year buckets, 30–34 through 80–84), % female slider, time horizon toggle (1 / 5 / 10 yr), CVD mortality reduction slider (0–60%).
- Right panel: KPI strip (4 cards) + tabbed Plotly charts (Survivors over time, Cumulative incidence) + Micro/Macro summary cards.
- Calls the simulation engine on every input change; no debounce needed given calculation speed.
- Reuses CSS variables and component classes from `heor-calculator/index.html` verbatim.

### Key Calculation Decisions

- **Age band blending**: use the midpoint age of the selected band as the starting age for life table lookup.
- **Sex blending**: blend male and female `qx` linearly by `pctFemale`.
- **CVD vs. all-cause mortality split**: CVD-attributable mortality fraction derived from incidence data; intervention reduces only CVD fraction; background mortality unchanged.
- **Life-years preserved**: `sum(survivors_intervention) - sum(survivors_baseline)` across all years, scaled to population size.
- **Micro survival gain**: `(survivors_intervention[horizon] - survivors_baseline[horizon]) / popSize * 100`, expressed as percentage points.

### Output Layout

```
KPI strip: [CVD Events (baseline)] [Deaths Avoided] [Life-years Preserved] [Survival Gain %]
Chart tabs: [Survivors over time] [Cumulative CVD Incidence]
Bottom row: [Micro card] [Macro card]
```

## Testing Decisions

**What makes a good test here**: test the simulation engine's outputs against known analytic values (e.g. zero mortality reduction → identical arms; 100% CVD mortality reduction → maximum possible deaths avoided bounded by CVD fraction). Do not test DOM rendering.

**Modules to test:**
- `getMortalityRate` / `getCVDIncidenceRate`: verify lookups return expected values for known age/sex/country combinations from the source data.
- Cohort simulation engine: verify that baseline and intervention arms are identical when `mortReduction = 0`; verify `deathsAvoided > 0` when `mortReduction > 0`; verify `lifeYearsPreserved` is non-negative.

**Prior art**: the existing `heor-calculator/index.html` uses inline JS with no test suite. For this calculator, the simulation engine should be extracted as a pure function to enable unit testing if a test harness is added to the project later.

## Out of Scope

- Multiple countries shown simultaneously (side-by-side comparison)
- Risk stratification / risk tier inputs
- Probabilistic sensitivity analysis (PSA) or confidence intervals
- Age distribution input (full distribution across age bands) — midpoint approximation used instead
- Export to PDF or Excel
- Server-side rendering or build pipeline — must remain a single static HTML file
- Finnish CVD incidence data (Finnish arm uses Swedish rates scaled by mortality ratio as an approximation)

## Further Notes

- The calculator lives at `cvd-simulator/index.html`, separate from `heor-calculator/index.html`.
- Life table source years: use the most recent available year in each dataset (Sweden: 2022; Finland: 2022 or closest available).
- The "Triple M" framing (Micro / Macro) should be visually distinct in the output — consider two highlighted summary cards below the charts, styled differently from the KPI strip to signal a narrative conclusion rather than a raw metric.
