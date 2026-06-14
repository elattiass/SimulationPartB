# Queuechella Simulation Project Context

## Purpose

This file is the working context for future changes to the Queuechella project.
It must describe the notebook as it actually exists, not only the intended design.

The project is a Simulation course assignment. Prefer simple, transparent,
course-aligned choices over advanced software engineering or statistical methods
that were not taught in the course.

When this file and the notebook disagree, inspect the notebook and the official
assignment before making changes, then update this file.

---

## Sources Of Truth

Use these sources in this order:

1. Official assignment: `Course Project 2026B.pdf`.
2. Active notebook: `Queuechella_Colab_Simulation_Report (4).ipynb`.
3. Project context: `context.md`.
4. Course summary material supplied by the user, when available.

An extracted Markdown copy of the assignment may also exist at:

```text
C:\Users\User\Downloads\Course Project 2026B.md
```

Older notebook copies `(1)`, `(2)`, `(3)`, and
`Queuechella_Colab_Simulation_Report.ipynb` are drafts unless the user explicitly
asks to use them.

---

## Current Deliverable

The official project deliverable is one notebook:

```text
Queuechella_Colab_Simulation_Report (4).ipynb
```

It contains both:

1. executable Python simulation code, and
2. the written simulation report.

The report uses Hebrew RTL formatting. Internal Python identifiers, dictionary
keys, metric names, scenario names, and event strings remain in English.

The notebook is designed to run locally with the project `.venv` using Python
3.11, and it remains compatible with Google Colab when the Excel file is uploaded
to the runtime.

Required data file:

```text
samples_for_simulation.xlsx
```

Do not hard-code a Windows path for the Excel file inside the notebook. The
notebook currently loads it relative to the kernel working directory.

---

## Course Modeling Principle

When choosing between a technically advanced solution and a simpler solution
based on the Simulation course, prefer the simpler course-aligned solution unless
the assignment explicitly requires additional complexity.

The notebook should clearly demonstrate:

- stochastic input modeling,
- generation from `U(0,1)`,
- inverse transform,
- composition,
- acceptance-rejection,
- Box-Muller,
- Bernoulli and Uniform sampling,
- event-oriented discrete-event simulation,
- queues and resources,
- replications,
- confidence intervals,
- relative precision,
- statistical comparison of alternatives,
- a recommendation under uncertainty.

Do not use high-level NumPy or SciPy random distribution calls for simulation
draws. SciPy may be used for fitting diagnostics, KS checks, and t critical
values.

Do not make the final model depend on advanced distributions or selection methods
that were not taught in the course, including Gamma, Weibull, Lognormal,
Triangular, Truncated Normal, or AIC-based model selection.

---

## Current Notebook Order

The active notebook currently follows this order:

1. title and project objective,
2. system description and assumptions,
3. discrete-event model structure,
4. event diagram,
5. imports, report helpers, and Excel loading,
6. input modeling,
7. sampling algorithms,
8. simulation classes,
9. validation checks,
10. current-state run,
11. current-state bottlenecks,
12. alternatives and budget,
13. pilot and final replications,
14. statistical comparison,
15. managerial decision rule,
16. final recommendation,
17. limitations and summary.

Preserve this executable order. Imports and `historical_data` must be defined
before the fitting and `Sampler` cells.

---

## Current Class Structure

The simulation implementation is divided into separate cells:

1. `Sampler`
2. `Event`
3. `EventCalendar`
4. `VisitorEntity`
5. `Resource`
6. `Stage`
7. `StatsCollector`
8. `SimulationConfig`
9. `QueuechellaSimulation`

Each simulation class has a short Hebrew Markdown explanation before its code
cell. Do not merge the classes back into one large cell unless explicitly asked.

`QueuechellaSimulation` owns the event loop, routing, queues, resources, stages,
state changes, and statistics. `Sampler` owns the course-level random-generation
methods. The simulation class may compose sampler methods when determining a
complete activity duration.

---

## Current Input Modeling

The workbook sheets used are:

```text
FriendsGroup_arrival_intervals
MainStage_concert_duration
```

Current final input models:

| Input | Model | Parameter source | Simulation sampling |
|---|---|---|---|
| FriendsGroup interarrival time | Exponential | rate estimated from the Excel sample | Inverse Transform |
| MainStage duration | Normal | mean and standard deviation estimated from the Excel sample | Box-Muller with a positive lower safeguard |

The fitting cell currently displays Exponential and Normal diagnostics for both
datasets, including log-likelihood and KS results. These diagnostics support the
course-level choices; they do not form an advanced automatic model-selection
pipeline.

---

## Current Sampler API

The active `Sampler` class uses static methods. Important methods are:

```text
u()
uniform(a, b)
bernoulli(p)
discrete_uniform(a, b)
exponential_inverse(mean)
discrete_choice_composition(weighted_items)
normal_box_muller(mean, std)
check_in_duration()
check_out_duration()
sample_friends_interarrival()
couple_interarrival_time()
single_interarrival_time()
sample_mainstage_duration()
destage_duration()
dj_density(x)
sample_dj_duration_acceptance_rejection()
photo_density(x)
sample_photo_duration()
merch_checkout_duration()
sample_battery()
sample_charging_duration(battery)
sample_bodyart_duration(art_type)
bar_service_time()
food_prep_duration(restaurant)
meal_consumption_duration()
sample_restaurant_choice()
sample_bodyart_type()
```

Use the names above when discussing or editing the current notebook. In
particular, the current PhotoStation method is named `sample_photo_duration()`,
not `sample_photo_station_service_time()`.

---

## Course Concepts Mapping

| Course method | Current use |
|---|---|
| Direct `U(0,1)` | base random number, Uniform, and Bernoulli logic |
| Inverse Transform | Exponential times and official PhotoStation service time |
| Composition | restaurant choice, body-art type, and other weighted discrete choices |
| Box-Muller | Normal MainStage duration and battery percentage |
| Acceptance-Rejection | DJStage stay duration |
| Discrete Uniform | FriendsGroup size from 3 to 6 |

---

## Official PhotoStation Sampling

PhotoStation service time is sampled from the official piecewise density using
inverse transform. The current method is:

```python
@staticmethod
def sample_photo_duration():
    u = Sampler.u()
    if u < 0.25:
        return math.sqrt(12.0 * u + 1.0)
    if u < 0.875:
        return (-5.0 + math.sqrt(281.0 + 640.0 * u)) / 8.0
    return 8.0 * u - 4.0
```

`photo_density(x)` remains in the class as supporting/explanatory code, but the
current PhotoStation sampler does not use acceptance-rejection.

Validation should confirm:

- every sample satisfies `1 <= X < 4`,
- approximately 0.25 of samples are in `[1,2)`,
- approximately 0.625 are in `[2,3)`,
- approximately 0.125 are in `[3,4)`.

Do not reintroduce the former `Uniform(3,7)` assumption or the old
acceptance-rejection implementation for PhotoStation.

---

## Current DJStage Sampling

DJStage stay duration uses acceptance-rejection on `[20,60]` with the official
piecewise density implemented by `dj_density(x)`.

The corrected density currently contains:

```python
if 20.0 <= x <= 40.0:
    return (x - 20.0) / 600.0
elif 40.0 < x <= 50.0:
    return ((60.0 - x) / 600.0) + (1.0 / 30.0)
elif 50.0 < x <= 60.0:
    return (60.0 - x) / 600.0
```

The proposal distribution is Uniform on `[20,60]`, and the implementation uses
`max_density = 1/15`.

---

## Current Event-Oriented Model

The event calendar is a priority queue ordered by `time` and `sequence`. Each
`Event` stores:

```text
time
sequence
event_type
entity_id
target
payload
```

The central event loop pops the next event, advances the simulation clock, and
dispatches dynamically to `handle_<event_type.lower()>`.

Current event strings include:

```text
ARRIVAL
ENTRANCE_SERVICE_END
ACTIVITY_DECISION
SERVICE_END
BODYART_BREAK_END
QUEUE_ABANDON
SHOW_START
SHOW_END
SHOW_EARLY_LEAVE
DJ_STAGE_EXIT
FOOD_ORDER_END
FOOD_PREP_END
FOOD_EATING_END
DAY_END
SIMULATION_END
```

Entrance and regular resource service starts are immediate helper operations,
not future-event-calendar event types. `try_start_next_resource()` calls
`start_entrance_service()` or `start_resource_service()` directly and schedules
the corresponding completion event.

`BODYART_BREAK_END` is a dedicated resource event. Every tenth completed
BodyArt service starts a 15-minute artist break; the artist remains unavailable
until this future event releases the blocked capacity and attempts to start the
next queued service.

MainStage and SideStage share the generic `SHOW_START` and `SHOW_END` event
strings. The relevant stage name is carried in the event `target`.

`SHOW_EARLY_LEAVE` currently applies to MainStage logic.

The DJ flow is:

```text
ACTIVITY_DECISION -> DJ_STAGE_EXIT -> ACTIVITY_DECISION
DJ_STAGE_EXIT -> DJ_STAGE_EXIT
```

DJStage admission is an immediate helper operation rather than a future event.
When the entity's full `group_size` fits, `start_dj_visit()` atomically adds that
guest count to `occupancy_guests` and schedules `DJ_STAGE_EXIT`. An exit may
immediately admit a queued entity and therefore schedule another future
`DJ_STAGE_EXIT`.

---

## Current Food Event Representation

Food uses three explicit completion events rather than `SERVICE_END` payload
phases. The current flow is:

```text
ACTIVITY_DECISION
-> immediate cashier-service start
-> FOOD_ORDER_END
-> FOOD_PREP_END
-> FOOD_EATING_END
-> ACTIVITY_DECISION
```

At `FOOD_ORDER_END`, the restaurant cashier is released and the next queued
visitor may begin ordering immediately. Payment and food satisfaction are then
processed, and food preparation is scheduled. The visitor remains in
`IN_SERVICE` state during preparation and eating, as in the previous
implementation. Food queues do not schedule queue-abandonment events.

Lunch eligibility is carried into `ACTIVITY_DECISION` only after completing a
regular station, a complete MainStage or SideStage performance, or DJStage.
Entrance completion, queue abandonment, early show departure, food completion,
and next-day startup schedule an ineligible activity decision. Each simulation
entity receives at most one shared Bernoulli lunch decision per festival day,
tracked in `lunch_decision_days`; a Couple or FriendsGroup is never split into
individual sub-visitors for this decision. The day is recorded before sampling,
so rejection cannot lead to repeated attempts. Eligible overnight entities may
receive a new decision on Day 2.

---

## Visitor Rules

| Entity | Current modeled behavior |
|---|---|
| FriendsGroup | arrives on Day 1 between 09:00 and 13:00; size is discrete Uniform 3-6; lodging probability 0.7; aims for one MainStage, one SideStage, one DJStage, then all stations |
| Couple | arrives on either day between 10:00 and 16:00; group size 2; does not visit DJStage; alternates between performances and stations; stops after five completed non-food activities; a qualified Day-1 couple may continue to Day 2 |
| Single | arrives on either day between 09:00 and 16:00; group size 1; visits Merch first; aims for two MainStage, two SideStage, and one DJStage visit |

Groups and couples move as one simulation entity, while satisfaction and revenue
are calculated at guest level where the code specifies.

Completed non-food activities are counted once in `completed_counts`. Food does
not count toward the Couple five-activity limit and does not change the previous
performance/station alternation state. An abandoned regular station continues
to count as one completed attempt and sets the last activity kind to `station`,
so the next choice preserves the required alternation.

---

## Day-End Rules

At 20:00 the simulation cleans regular resource queues, stage queues, active
resource users, and stage attendees.

- Singles must depart.
- FriendsGroup may continue only when lodging is true.
- Couples may continue only after Day 1 when they arrived on Day 1 and average
  satisfaction is greater than 7.
- Non-eligible visitors are removed from all active locations and departed.
- Eligible visitors have transient location state cleared and receive a normal
  `ACTIVITY_DECISION` event at the next day start.

`DAY_END` leads to next-day activity only for eligible visitors. The simulation
terminates at `SIMULATION_END` after the second operating day.

---

## Current Scenarios And Budget

Scenario names must remain exactly:

```text
Current
Combo A
Combo B
Combo C
```

Budget limit:

```text
1,000,000 NIS
```

| Scenario | Cost | Current parameter changes |
|---|---:|---|
| Current | 0 | baseline |
| Combo A | 950,000 | automatic entrance scan; PhotoStation capacity 4; BodyArt capacity 3; initial satisfaction 6.5 |
| Combo B | 900,000 | automatic entrance scan; MainStage genre weight 4 |
| Combo C | 850,000 | arrivals x1.2; PhotoStation capacity 4; BodyArt capacity 3; MainStage genre weight 4; initial satisfaction 6.5 |

Scenario effects should remain parameter-driven through `SimulationConfig` unless
the assignment or user explicitly requests a modeling change.

---

## Current Statistical Workflow

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

Current workflow:

1. Run `PILOT_REPLICATIONS = 8` with common scenario seeds.
2. Estimate the required number of replications using t-based confidence
   intervals.
3. Select the maximum required n across scenarios and primary metrics.
4. Run all scenarios again to create `final_results`.
5. If any primary metric has relative precision above 0.1, increase n and rerun
   until the target is met.
6. Build final summaries and paired differences versus Current.
7. Generate the recommendation from `final_results`, not pilot results.

Parameters:

```text
confidence level = 0.90
relative precision target = 0.10
overall alpha = 0.10
```

Paired comparisons use matched replication numbers/seeds. There are 3
alternatives x 3 primary metrics = 9 simultaneous comparisons, so:

```text
alpha_i = 0.10 / 9
```

Each paired-comparison row includes:

```text
n_pairs
paired_mean_diff
paired_sd_diff
df
alpha_i
t_crit
margin_error
ci_low
ci_high
significant_bonferroni
```

If common random-number pairing is broken, do not silently retain the paired
comparison.

---

## Current Recommendation Logic

The recommendation cell calls:

```python
final_metric_records(final_results)
```

It separately identifies:

1. the best visitor-experience alternative,
2. the best waiting-time alternative,
3. the best short-term net-profit option.

Do not base the final recommendation on `pilot_results`.

---

## Current Validation Coverage

The validation section currently checks:

- nondecreasing event-calendar order,
- DJ acceptance-rejection support,
- finite Box-Muller output,
- PhotoStation support and interval proportions,
- distinct food order, preparation, and eating events with immediate cashier reuse,
- one shared lunch decision per entity per day and a fresh eligible Day-2 decision,
- lunch eligibility only after completed stations and performances,
- Couple completion counting against exactly five non-food activities,
- food preserving the Couple's previous non-food alternation state,
- abandoned regular stations counting once and updating Couple alternation,
- dedicated `BODYART_BREAK_END` scheduling and artist release,
- guest-sized immediate DJStage admission, atomic queue refill, and day-end cleanup,
- day-end queue/resource/stage cleanup,
- FriendsGroup, Couple, and Single arrival windows,
- FriendsGroup size,
- satisfaction bounds,
- resource and stage capacity sanity,
- itinerary limits,
- no Couple DJStage completion,
- scenario budget compliance,
- positive visitors and nonnegative waiting time in a smoke run.

After any simulation-code change:

1. compile every code cell,
2. run the notebook top-to-bottom,
3. confirm no saved error outputs remain,
4. confirm validation checks pass,
5. confirm `final_results` exists,
6. confirm final precision reaches 0.1,
7. confirm the recommendation still uses `final_results`.

---

## Report Formatting

Use clean academic Colab formatting:

- Hebrew RTL explanations,
- compact tables and plots,
- formulas where course justification is needed,
- no retro-terminal or product-dashboard language.

Code comments must remain in English.

Do not translate code-dependent names such as class names, function names,
dictionary keys, metric names, scenario names, or event strings. Hebrew labels
may be created for display-only tables.

---

## Git And Merge Safety

Do not assume the current branch from this file. Check `git status` before work.

The notebook has received merged contributions from multiple branches. Preserve
the latest merged `main` content together with the corrected PhotoStation and
DJStage samplers.

Notebook merge conflicts should not be resolved by blindly accepting an entire
side. Compare notebook cells, preserve substantive changes, restore executable
cell order, run top-to-bottom, and only then stage the resolved notebook.

Do not commit `.venv`, temporary debug files, old notebook drafts, or secrets.

---

## Security

Do not store API keys, tokens, passwords, or credentials in notebooks or tracked
project files. Use environment variables or the operating system credential
manager if credentials are ever required.

---

## Working Procedure For Future Changes

Before editing:

1. inspect the active notebook and `git status`,
2. identify the exact affected cells,
3. distinguish current implementation from proposed future design,
4. preserve internal identifiers and event strings unless the change explicitly
   requires updating them,
5. make the smallest targeted change,
6. update diagrams and context when event logic changes,
7. run the notebook and verify statistical outputs after behavioral changes.

Do not rebuild the notebook from scratch unless explicitly requested.
