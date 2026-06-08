# Queuechella Simulation Project Context

## Purpose Of This File

This file is the working context for the Queuechella project. It should guide every future coding or report-editing step in this repository.

The project must read as a **Simulation course assignment**, not as a professional software product. Prefer simple, transparent, course-aligned modeling choices over technically fancy solutions.

Expanded course material is available locally at:

```text
c:\Users\User\Documents\לימודים\שנה ג'\סמ' ב\סימולציה\simulation_course_context.md
```

Use that file as the detailed course reference when needed. This project-level `context.md` summarizes the rules that matter most for the Queuechella notebook.

---

## Current Main Deliverable

The official deliverable notebook is:

```text
Queuechella_Colab_Simulation_Report (4).ipynb
```

Older notebook copies such as `(1)`, `(2)`, `(3)`, and `Queuechella_Colab_Simulation_Report.ipynb` are draft copies and should not be treated as the active deliverable unless the user explicitly asks.

The notebook is both:

1. executable simulation code, and
2. the written project report.

The report is currently written mainly in **Hebrew RTL academic style**. Internal Python identifiers remain in English.

---

## Current Repository / Git Notes

- The project was pushed to GitHub at:

```text
https://github.com/elattiass/SimulationPartB.git
```

- Branch used locally: `Elad`.
- `Queuechella_Colab_Simulation_Report (4).ipynb` currently has local changes after the last push.
- If asked to push again, commit the current notebook and `context.md` changes deliberately.
- Do not include old draft notebooks unless the user explicitly requests it.

---

## Core Course Principle

When choosing between:

- a technically advanced solution, and
- a simpler solution based on the Simulation course tools,

choose the **simpler course-aligned solution**, unless the assignment explicitly requires more complexity.

Do not invent new methods that were not taught or are not required.

The model should demonstrate understanding of:

- stochastic input modeling,
- sampling from `U(0,1)`,
- discrete-event simulation,
- event calendars,
- queues and resources,
- replications,
- confidence intervals,
- relative precision,
- statistical comparison of alternatives,
- final recommendation under uncertainty.

---

## Allowed Course Distributions And Sampling Methods

Use only course-level distributions and methods unless the assignment explicitly gives a special formula.

### Allowed distributions

- Continuous Uniform `U(a,b)`
- Discrete Uniform
- Bernoulli
- Discrete / categorical distributions
- Exponential
- Normal
- Empirical distribution, if needed
- Official piecewise densities from the assignment, when specified

### Required sampling methods from `U(0,1)`

| Course method | Use in Queuechella |
|---|---|
| Direct `U(0,1)` | base random numbers, Uniform, Bernoulli |
| Inverse Transform | Exponential interarrival times, PhotoStation official piecewise service time |
| Composition | discrete choices such as body art, food, routing options |
| Box-Muller | Normal sampling |
| Acceptance-Rejection | DJStage stay duration |
| Empirical / categorical sampling | only if needed and implemented through cumulative probabilities |

Do not use high-level NumPy/SciPy random distribution calls for simulation draws. SciPy may be used for simple goodness-of-fit checks and statistical critical values.

Avoid advanced fitting language and advanced distributions not taught in the course. In particular, do not make the final model depend on:

```text
Gamma, Weibull, Lognormal, Triangular, Truncated Normal, AIC-based model selection
```

---

## Course Concepts That Should Guide Work

From the attached course context, the most relevant concepts are:

### Simulation study process

1. define the objective,
2. understand the system,
3. define entities, resources, state variables and events,
4. build event flow and pseudocode,
5. implement,
6. verify code,
7. validate model assumptions,
8. run replications,
9. compute confidence intervals and precision,
10. compare alternatives statistically,
11. recommend under uncertainty.

### Discrete-event simulation

The model is a DES because state changes occur at discrete event times:

- arrivals,
- service starts,
- service completions,
- queue abandonment,
- show starts,
- show ends,
- DJ entry/exit,
- day end,
- simulation end.

Between events, the state is unchanged.

### Event calendar

Use a priority queue ordered by event time. Each event should contain:

```text
time
sequence
event_type
entity_id
target
payload
```

### Five-step event-programming style

For each major event:

1. identify the event trigger,
2. update the simulation clock,
3. update state variables,
4. update statistical accumulators,
5. schedule future events.

### Verification and validation

- Verification: code works as intended.
- Validation: model assumptions are reasonable for the assignment.

The notebook should keep validation checks visible and explain what they confirm.

---

## Current Notebook Architecture

The current notebook uses:

- one central `Sampler` class for model-specific random sampling,
- split code cells for simulation classes,
- `QueuechellaSimulation` as the central event engine,
- a calendar-driven event loop,
- scenario definitions through `SimulationConfig`,
- final statistical comparison from `final_results`.

### Split class structure

The "Current system implementation" section has been split into smaller notebook cells:

1. `Event`
2. `EventCalendar`
3. `VisitorEntity`
4. `Resource`
5. `Stage`
6. `StatsCollector`
7. `SimulationConfig`
8. `QueuechellaSimulation`

Each class cell now has a short Hebrew Markdown explanation before it.

Do not merge these back into one large code cell unless the user explicitly asks.

### Important design rule

All sampling formulas should live in `Sampler`.

`QueuechellaSimulation` should consume the sampler and focus on:

- event handling,
- state changes,
- routing,
- queues,
- resource/stage logic,
- statistics.

---

## Current Sampling Implementation

The notebook currently centralizes sampling in `Sampler`.

Important methods include:

```text
u()
uniform(a, b)
bernoulli(p)
discrete_uniform(a, b)
exponential_inverse(mean)
normal_box_muller(mean, std)
discrete_choice_composition(weighted_items)
sample_friends_interarrival()
sample_mainstage_duration()
sample_dj_duration_acceptance_rejection()
sample_battery()
sample_charging_duration(battery)
sample_photo_station_service_time()
sample_entrance_service_time(entrance_auto_scan)
sample_merch_service_time()
sample_body_art_service_time()
sample_food_service_time(activity)
sample_side_stage_duration()
```

Do not move sampling formulas back into `QueuechellaSimulation`.

---

## Current Input Modeling

The notebook uses the workbook:

```text
samples_for_simulation.xlsx
```

Required sheets:

1. `FriendsGroup_arrival_intervals`
2. `MainStage_concert_duration`

Current final input models:

| Input | Final course-level model | Sampling method |
|---|---|---|
| FriendsGroup interarrival intervals | Exponential, mean from Excel sample | Inverse Transform |
| MainStage concert duration | Normal, mean/std from Excel sample | Box-Muller with positive safeguard |

Goodness-of-fit checks may be shown simply, but do not turn this into an advanced auto-fitting pipeline.

---

## Official PhotoStation Sampling

PhotoStation service time is no longer an assumption.

It is sampled from the official assignment piecewise density using inverse transform.

The current sampler is:

```python
def sample_photo_station_service_time(self):
    u = self.u()
    if u < 0.25:
        return math.sqrt(12.0 * u + 1.0)
    if u < 0.875:
        return (-5.0 + math.sqrt(281.0 + 640.0 * u)) / 8.0
    return 8.0 * u - 4.0
```

Validation currently samples 10,000 values and checks:

- all values are in `[1,4)`,
- interval proportions approximately match:
  - `[1,2)`: `0.25`
  - `[2,3)`: `0.625`
  - `[3,4)`: `0.125`

Never reintroduce the old `Uniform(3,7)` PhotoStation assumption.

---

## Current Event And Visitor Logic

Keep event strings in English exactly as implemented. Do not translate internal event names.

Important event strings include:

```text
ARRIVAL
ENTRANCE_SERVICE_START
ENTRANCE_SERVICE_END
ACTIVITY_DECISION
QUEUE_ABANDON
SERVICE_START
SERVICE_END
MAIN_STAGE_SHOW_START
MAIN_STAGE_SHOW_END
SIDE_STAGE_SHOW_START
SIDE_STAGE_SHOW_END
SHOW_EARLY_LEAVE
DJ_STAGE_ENTER
DJ_STAGE_EXIT
DAY_END
SIMULATION_END
```

### Visitor types

| Entity type | Key behavior |
|---|---|
| FriendsGroup | Day 1 only, size 3-6, lodging probability 0.7, one MainStage, one SideStage, one DJStage, then all stations |
| Couple | Day 1 or 2, 10:00-16:00, no DJStage, alternates between performances and stations, may continue to Day 2 if satisfaction > 7 |
| Single | Day 1 or 2, one-day visitor, Merch first, then 2 MainStage, 2 SideStage, 1 DJStage |

### Day-end cleanup

At 20:00:

- Singles must depart.
- FriendsGroup may continue only if lodging.
- Couples may continue only if Day 1 arrival and satisfaction > 7.
- Non-eligible visitors are removed from queues, resources and stages.
- Eligible visitors resume on Day 2 through the normal `ACTIVITY_DECISION` mechanism.

Do not remove or weaken this cleanup.

---

## Current Alternatives

Current scenarios must remain named exactly:

```text
Current
Combo A
Combo B
Combo C
```

Current combinations:

| Scenario | Cost | Main changes |
|---|---:|---|
| Current | 0 | baseline |
| Combo A | 850,000 | better kitchen staff, extra photo/body art capacity, visitor benefit |
| Combo B | 950,000 | expanded stage capacity, popular MainStage bands |
| Combo C | 950,000 | automatic ticket scanning, advertising, extra photo/body art capacity |

Budget constraint:

```text
1,000,000 NIS
```

Alternatives should be implemented only through `SimulationConfig` parameters unless the user explicitly asks for a modeling change.

---

## Current Statistical Analysis

The notebook uses:

- pilot replications,
- required-n calculation,
- final replications,
- final summary tables,
- paired differences versus Current,
- final recommendation from `final_results`.

### Confidence intervals and precision

Use t-based confidence intervals for finite replications.

Relative precision target:

```text
0.1
```

Overall confidence level:

```text
0.9
```

### Alternative comparisons

The final paired comparison uses matched replications/seeds:

```text
d_j = metric_alternative_j - metric_current_j
```

There are:

```text
3 alternatives × 3 primary metrics = 9 comparisons
```

Use Bonferroni:

```text
alpha_i = 0.10 / 9
```

For each metric/alternative report:

- `n_pairs`
- `paired_mean_diff`
- `paired_sd_diff`
- `df`
- `alpha_i`
- `t_crit`
- `margin_error`
- `ci_low`
- `ci_high`
- `significant_bonferroni`

Display significance in Hebrew as:

```text
מובהק לאחר Bonferroni = כן / לא
```

If matching by replication/seed is ever broken, do not silently use paired t. Use Welch comparison or redesign CRN.

---

## Primary And Secondary Metrics

Primary metrics:

```text
avg_satisfaction
avg_wait_all
net_profit
```

Secondary metrics:

```text
visitor_guests
total_abandonments
total_revenue
```

Recommendation currently separates:

1. best visitor-experience alternative,
2. best waiting-time alternative,
3. best short-term profit option.

The recommendation must remain based on `final_results`, not pilot results.

---

## Report Formatting Direction

Use clean academic Colab formatting.

Current report language/style:

- Hebrew explanations,
- RTL Markdown/HTML,
- compact tables,
- course-report tone,
- no retro/terminal/product-dashboard style.

Internal code names stay English.

Do not translate:

- class names,
- function names,
- dictionary keys,
- DataFrame columns used by code,
- scenario names,
- event strings.

Display labels may be Hebrew.

---

## Validation Expectations

Keep validation tests visible and runnable.

Current validation covers:

- event calendar ordering,
- sampler range checks,
- PhotoStation inverse-transform checks,
- day-end cleanup,
- arrival windows,
- capacity sanity,
- satisfaction bounds,
- itinerary rules,
- budget constraints,
- smoke run event log.

After any notebook code change:

1. compile all code cells,
2. run the notebook top-to-bottom,
3. confirm validation checks pass,
4. confirm `final_results` exists,
5. confirm recommendation uses `final_results`.

---

## Security Rules

Do not store secrets, API keys, tokens or credentials in notebooks, frontend files, public config files or committed code.

If credentials are ever needed, use local environment variables or GitHub credential manager, not hardcoded values.

---

## Working Style For Future Changes

Before changing code:

1. inspect the current notebook,
2. identify the exact cell(s),
3. preserve internal names and event strings,
4. make the smallest targeted change,
5. run the notebook when code behavior changes.

Prefer notebook-structure edits through JSON-aware scripts when splitting or inserting cells.

For text-only report changes, keep Hebrew RTL style consistent.

For code comments, use English only.

Do not rebuild the notebook from scratch unless the user explicitly asks.
