# context.md


## Source Instruction File

The official project instructions file is named: **Course Project 2026B**.

This file is the authoritative assignment source for the Queuechella simulation requirements, deliverables, constraints, and grading expectations.


## Project Identity

**Course Project 2026B** is a discrete-event simulation project for the Simulation course.  
The simulated system is the **Queuechella music festival**, a two-day festival that wants to evaluate its current daily operation, identify bottlenecks, and test improvement alternatives under a fixed budget.

The coding agent must treat this project primarily as a **simulation logic, event mapping, and course-method demonstration task**, not as a quick UI or code-generation task.

---

## Course-Style Modeling Principle

The notebook should prioritize simple, transparent, course-based modeling choices. The goal is to demonstrate understanding of the Simulation course material, not to build an advanced professional simulation product.

When choosing between a technically advanced solution and a simpler course-aligned solution, prefer the simpler course-aligned solution unless the assignment explicitly requires complexity.

---

## Core Course Objective

The course expects students to use simulation to model real-world processes with stochastic behavior, discrete events, input modeling, experiment design, and output analysis.

The project must demonstrate:

- Distribution fitting from historical data.
- Custom random sampling algorithms from `U(0,1)`.
- Object-oriented modeling in Python.
- Modular, readable, top-down code.
- Event-oriented simulation logic.
- Statistical comparison between the current state and improvement alternatives.
- Logical and statistical justification for recommendations.

The report should make each modeling choice easy to explain in an oral defense. Prefer clear assumptions, compact formulas, and direct links between course concepts and implemented code.

---

## Required Deliverables

The final academic deliverable is one **Google Colab notebook** that includes both code and report text.

The notebook must include:

1. Simulation code blocks.
2. English-only code comments.
3. Written report sections around the code:
   - Introduction.
   - System description.
   - Flow/event diagrams.
   - Distribution fitting explanations.
   - Mathematical formulas.
   - Sampling algorithm explanations.
   - Event-oriented implementation logic.
   - Assumptions.
   - Alternative selection and implementation.
   - Required number of simulation replications.
   - Statistical comparison between alternatives.
   - Recommendations.
   - Summary.
4. Event diagram before implementing the current state.
5. Detailed handling diagrams for three selected events.
6. Final presentation of results and recommendations.
7. Full code understanding for oral defense.

Deadline mentioned in the assignment: **29.06.2026**.

---

## Technical Constraints

### Programming Environment

- Primary implementation environment: **Google Colab**.
- Language: **Python**.
- Code style: **Object-Oriented**, **Top-Down**, modular, efficient, and readable.
- Code comments must be in **English only**.

### Random Sampling Requirements

All random values must be generated through course-taught sampling methods using random numbers from `U(0,1)`.

Allowed/custom-required sampling methods include:

- Inverse transform.
- Composition.
- Acceptance-rejection.
- Box-Muller for normal distributions.
- Custom distribution fitting based on provided historical data.

Do not rely blindly on built-in high-level distribution samplers when the assignment expects manual sampling logic.

Prefer simple course-level distributions as the main modeling choices whenever they are reasonable:

- Exponential.
- Uniform.
- Normal.
- Bernoulli.
- Discrete distributions.

Advanced distributions may be used only as secondary diagnostic candidates when helpful, not as the center of the final model unless the assignment data clearly requires them.

---

## Course Concepts Mapping

The notebook should explicitly connect implemented logic to Simulation course methods:

| Course concept | Queuechella use |
|---|---|
| Inverse Transform | Exponential interarrival times and other invertible service-time models |
| Box-Muller | Normal sampling, such as battery percentage and normal service components |
| Acceptance-Rejection | DJStage stay duration from the custom piecewise density |
| Composition | Discrete activity, restaurant, body-art, and routing choices |
| Bernoulli | Yes/no decisions such as purchases, satisfaction outcomes, lodging, and abandonment-related decisions |
| Uniform | Simple bounded service times and discrete uniform group size |

### Historical Data File

The uploaded workbook `samples_for_simulation.xlsx` contains two sheets:

1. `FriendsGroup_arrival_intervals`
   - One column: `minutes`
   - Represents historical arrival interval data for FriendsGroup arrivals.
   - Must be fitted to a suitable distribution and sampled from through a custom sampling function.

2. `MainStage_concert_duration`
   - One column: `minutes`
   - Represents historical MainStage concert duration data.
   - Must be fitted to a suitable distribution and sampled from through a custom sampling function.

---

## Simulation Time Scope

The festival lasts **two days**.

Each day operates from:

- Start: `09:00`
- End: `20:00`

Recommended internal simulation time unit: **minutes from Day 1 at 09:00**.

Example:

- Day 1, 09:00 = `0`
- Day 1, 20:00 = `660`
- Day 2, 09:00 = `1440`
- Day 2, 20:00 = `2100`

The model should support event scheduling across both days.

---

## Main System Objective

The simulation must first model the **current state** of Queuechella.

After the current state works, the model must test improvement alternatives under a total budget constraint of:

```text
1,000,000 NIS
```

The team must choose **2-3 performance metrics** and evaluate at least **two combinations of alternatives** against the current state.

Recommendations must be justified:

- Statistically.
- Logically.
- Based on the selected performance metrics.

---

## Suggested Performance Metrics

The assignment allows choosing the metrics. Recommended metrics for this project:

1. **Average visitor satisfaction**
   - Overall and by visitor type.

2. **Average queue waiting time**
   - Per facility/activity.
   - Especially entrance, food, photo station, body art, and stages.

3. **Total festival profit or revenue**
   - Tickets.
   - Lodging.
   - Merch.
   - Food.
   - Photo purchases.
   - Minus alternative costs, if comparing business value.

Optional secondary metrics:

- Abandonment count.
- Stage utilization.
- Food dissatisfaction count.
- Number of visitors served.
- Percentage of visitors with satisfaction below threshold.
- Revenue per visitor.

---

## Entities

### Base Entity: VisitorEntity

A visitor entity is a simulation object that moves through the festival as a unit.

Important distinction:

- **Guest / person**: one human visitor.
- **Entity**: a simulation unit, such as a single, couple, or group.

A group or couple moves together. No member continues to the next activity until the whole entity finishes the current activity.

Recommended attributes:

```text
entity_id
entity_type
group_size
arrival_day
arrival_time
satisfaction_per_guest
current_activity
planned_activities
completed_activities
is_lodging
queue_enter_time
total_wait_time
total_spend
abandoned_queues_count
```

---

## Visitor Types

### FriendsGroup

Rules:

- Group size: discrete uniform `[3, 6]`.
- Arrival rate: based on the historical Excel sheet.
- Arrives only on Day 1.
- Arrival window: `09:00-13:00`.
- Probability of staying overnight: `0.7`.
- Wants to see one performance of each genre:
  - Mainstream.
  - Indie.
  - Electronic.
- After full attendance in each performance type, visits all festival stations.
- Chooses stations by shortest queue.

### Couple

Rules:

- Arrival process: exponential with mean `60` per hour.
- Arrival window: `10:00-16:00`.
- May arrive on either day.
- If arrived on Day 1 and satisfaction is above `7` at end of day, stays overnight.
- Alternates between performances and stations.
- Does not like electronic music.
- Chooses next activity with equal probability among available activities.

### Single

Rules:

- Arrival process: exponential with mean `500` per day.
- Arrival window: `09:00-16:00`.
- Can arrive on Day 1 or Day 2.
- One-day visitor only.
- First goes directly to MerchTent.
- Then attends:
  - 2 MainStage shows.
  - 2 SideStage shows.
  - 1 DJStage stay.
- Prioritizes the shortest queue.

---

## Satisfaction Model

Each guest starts with:

```text
initial satisfaction = 5
minimum satisfaction = 0
maximum satisfaction = 10
```

Satisfaction updates throughout the simulation.

All updates must be clipped to `[0, 10]`.

### Queue Abandonment Penalties

Visitors have maximum waiting tolerance for non-show stations:

| Entity Type | Max Queue Wait | Satisfaction Penalty |
|---|---:|---:|
| FriendsGroup | 15 minutes | -2 |
| Couple | 20 minutes | -1.5 |
| Single | 20 minutes | -1 |

### Performance Satisfaction

After watching a show, each guest has:

- Probability `0.5`: good experience.
- Probability `0.5`: bad experience.

Good experience score increase:

```text
score = ((G - 1) / 2) + ((T - 1) / 19)
```

Where:

```text
G = genre weight
Mainstream = 3
Indie = 2
Electronic = 1
T = hour when the show ends
```

Bad experience:

```text
satisfaction -= 1
```

---

## Activities and Resources

### Entrance

Every arriving entity must pass the entrance before entering the festival.

Resources:

- 5 clerks.

Service process:

```text
ticket scan duration ~ Uniform(1.5, 3)
security check duration ~ Exponential(mean=2)
total entrance service = scan duration + security duration
```

Ticket and lodging prices:

```text
entry ticket = 500 NIS
lodging add-on = 250 NIS
entry + lodging bundle = 700 NIS
```

### MainStage

Purpose:

- Mainstream performances.

Rules:

- Concert duration: fitted from the Excel sheet `MainStage_concert_duration`.
- Break between concerts: 10 minutes.
- Capacity: 200 guests per show.
- Entry order determines position.
- The 10 farthest entities may leave 15 minutes after entering with probability `0.5`.
- If space opens during a show, waiting entities may enter.
- Maximize occupancy when possible:
  - If only one seat remains and the first waiting entity is a couple, a single behind them may enter instead.
  - If no fitting entity exists, the show may run below full capacity.

### SideStage

Purpose:

- Indie performances.

Rules:

```text
duration ~ Uniform(20, 30)
break between shows = 5 minutes
capacity = 100 guests
```

### DJStage

Purpose:

- Electronic music.
- Operates continuously during festival hours.

Rules:

```text
capacity = 70 guests at any time
stay duration follows a custom piecewise density
sampling method required: acceptance-rejection
```

Piecewise duration density:

```text
f(x) =
(x - 20) / 600                 for 20 <= x <= 40
(60 - x) / 600 + 1/30          for 40 <= x <= 50
(60 - x) / 600                 for 50 <= x <= 60
0                              otherwise
```

### PhotoStation

Resources:

```text
3 photo stations
shared queue
```

Rules:

- Photo duration distribution must be implemented according to the assignment data/formula.
- Probability `0.7`: visitor is satisfied with the photo.
  - satisfaction `+2`
  - buys printed copy for `30 NIS`
- If not satisfied:
  - probability `0.5`: satisfaction `-0.5`

### ChargingStation

Resources:

```text
150 chargers
```

Battery on arrival:

```text
b ~ Normal(mean=40, std=15)
```

Charging duration:

```text
f(t) = α / 40^α * (40 - t)^(α - 1), 0 <= t <= 40
α = 100 / (100 - b)
```

### MerchTent

Resources:

```text
7 cashiers
```

Purchase duration:

```text
Uniform(2, 6)
```

Purchase probabilities per guest:

```text
Festival shirt: probability 0.8, price 100 NIS
Festival hat: probability 0.4, price 50 NIS
Flag: probability 0.9, price 40 NIS
Band shirt: probability 0.3, price 200 NIS
```

### BodyArt

Resources:

```text
2 artists
```

Body art choices:

| Choice | Probability | Satisfaction Probability | Satisfaction Increase | Duration |
|---|---:|---:|---:|---|
| Glitter + gems | 0.3 | 0.7 | +0.8 | Normal(15, 3) |
| Neon paint | 0.3 | 0.6 | +1.2 | Exponential(mean=12) |
| Henna tattoo | 0.4 | 0.8 | +0.7 | Uniform(17, 22) |

Artist break rule:

```text
Each artist takes a 15-minute break after every 10 completed artworks.
```

### Food Court

Lunch decision:

- After completing an activity between `13:00-15:00`, each guest may choose lunch.
- `70%` of visitors choose to eat lunch.

Restaurants:

| Restaurant | Preference | Preparation Time | Price |
|---|---:|---|---:|
| Hamburger | 3/8 | Uniform(3, 4) | 100 NIS |
| Pizza | 1/4 | Uniform(4, 6) | Personal: 40 NIS, Family tray: 100 NIS |
| Asian | Remaining probability | Uniform(3, 7) | 65 NIS |

Additional food rules:

- Each restaurant has one cashier.
- Order/payment service duration:
  - Normal(mean=5, std=1.5)
- Eating duration:
  - Uniform(15, 35)
- Probability `0.4`: visitor is dissatisfied with food.
  - satisfaction `-0.6`
- Singles order only personal pizza.
- Family pizza tray is enough for 3 people.

---

## Event-Oriented Simulation Implementation

The project must be implemented as a discrete-event simulation. The event calendar, object-oriented classes, entities, resources, queues, state variables, and statistics should be presented as a readable implementation of course DES concepts for explanation and defense.

The purpose of the structure is clarity: each event changes the system state, updates statistics, and schedules future events. It should not be described as professional software architecture or an advanced simulation platform.

Recommended simulation loop:

```text
1. Initialize simulation clock.
2. Initialize system state.
3. Initialize event calendar.
4. Schedule initial arrivals and first stage events.
5. Pop the next event from the event calendar.
6. Advance the simulation clock.
7. Execute event handler.
8. Update state, queues, resources, statistics, and future events.
9. Continue until the stopping condition.
10. Produce output metrics.
```

Recommended event calendar:

```text
PriorityQueue ordered by event_time
```

Recommended event object:

```text
event_id
event_time
event_type
entity_id
resource_id
payload
```

---

## Core Event Types

At minimum, the model should define handlers for:

```text
ARRIVAL
ENTRANCE_SERVICE_START
ENTRANCE_SERVICE_END
ACTIVITY_DECISION
QUEUE_JOIN
QUEUE_ABANDON
SERVICE_START
SERVICE_END
SHOW_START
SHOW_ENTRY
SHOW_EARLY_LEAVE
SHOW_END
LUNCH_DECISION
DAY_END
SIMULATION_END
```

Possible stage-specific events:

```text
MAIN_STAGE_SHOW_START
MAIN_STAGE_SHOW_END
SIDE_STAGE_SHOW_START
SIDE_STAGE_SHOW_END
DJ_STAGE_ENTER
DJ_STAGE_EXIT
```

---

## Recommended Data Model

The following objects are recommended because they make the discrete-event simulation easier to read, test, and defend. Keep them simple and focused on the assignment logic.

### SimulationConfig

Stores all configurable parameters:

```text
operating_hours
capacities
service_rates
distribution_parameters
prices
alternative_flags
budget
random_seed
replication_count
```

### SimulationState

Stores mutable state:

```text
clock
event_calendar
entities
resources
queues
active_shows
statistics
daily_state
```

### Resource

Generic service resource:

```text
resource_id
resource_name
capacity
busy_units
queue
service_policy
```

### Stage

Specialized resource for performances:

```text
stage_id
genre
capacity
current_show
queue
break_duration
duration_sampler
```

### StatsCollector

Tracks simulation outputs:

```text
wait_times
queue_lengths
abandonments
satisfaction_values
revenue
utilization
visitor_counts
```

### Sampler

Central location for all random sampling methods:

```text
uniform()
bernoulli(p)
discrete_uniform(a, b)
exponential(mean)
normal_box_muller(mean, std)
inverse_transform(...)
composition(...)
acceptance_rejection(...)
sample_fitted_friends_arrival()
sample_fitted_mainstage_duration()
```

---

## Distribution Fitting Tasks

The coding agent must not skip input modeling.

Required distribution-fitting work:

1. Load historical data from the Excel workbook.
2. Visualize or summarize data.
3. Test simple course-level candidate distributions first.
4. Estimate parameters.
5. Use goodness-of-fit reasoning.
6. Implement custom sampling functions based on the selected distributions.

At minimum:

```text
FriendsGroup_arrival_intervals -> fitted interarrival distribution
MainStage_concert_duration -> fitted show duration distribution
```

Recommended fitting workflow:

```text
inspect histogram
estimate parameters
compare simple candidate distributions
run goodness-of-fit test where appropriate
document selected distribution
implement sampler
validate sampled values visually/statistically
```

Preferred final-model candidates:

- Exponential.
- Uniform.
- Normal.
- Bernoulli.
- Discrete distributions.

Secondary diagnostic candidates may include Weibull, Gamma, Lognormal, Triangular, or Truncated Normal, but these should not make the project look like an advanced automatic fitting pipeline. If they are mentioned, explain that they were considered only for comparison and choose the simplest defensible course-aligned model whenever possible.

---

## Improvement Alternatives

The simulation must compare at least two combinations of alternatives under the budget.

Budget limit:

```text
1,000,000 NIS
```

Available alternatives:

| Alternative | Cost | Effect |
|---|---:|---|
| Better kitchen staff | 500,000 | Food dissatisfaction probability decreases to 0.1; lunch participation rises to 0.85 |
| Expanded security staff | 650,000 | Capacity of all stage areas increases by 30% |
| Popular MainStage bands | 300,000 | Band shirt purchase probability rises to 0.8; Mainstream genre weight increases from 3 to 4 |
| Extra photo station + extra body artist | 150,000 | Photo stations increase from 3 to 4; body artists increase from 2 to 3 |
| Festival advertising | 200,000 | Visitor arrival rate increases by 20% |
| Automatic ticket scanning stations | 600,000 | Entrance service becomes security-check time only |
| Visitor benefit | 200,000 | Initial satisfaction starts at 6.5 |

Recommended alternative-combination examples:

```text
Combo A: Better kitchen staff + Extra photo/body art + Visitor benefit
Cost: 850,000

Combo B: Expanded security staff + Popular MainStage bands
Cost: 950,000

Combo C: Automatic ticket scanning + Advertising + Extra photo/body art
Cost: 950,000
```

The final choice should be based on simulation outputs, not intuition alone.

---

## Statistical Output Analysis

After implementing the current state and alternatives, the model must calculate the required number of replications and compare alternatives statistically.

Required comparison target:

```text
general confidence level = 0.9
relative precision = 0.1
```

Expected analysis topics:

- Warm-up consideration if relevant.
- Replication count estimation.
- Paired t-test or other course-approved comparison methods.
- Multiple-alternative comparison if needed.
- Confidence intervals.
- Statistical and logical explanation of recommendation.

---

## Report Formatting Direction

Use clean academic Google Colab report formatting. Tables and charts should support explanation, validation, and defense of the simulation model, not look like a product interface.

Recommended report outputs:

- Clear markdown headings.
- Compact tables for parameters, events, queues, resources, alternatives, and results.
- Simple plots for historical data, fitted distributions, and scenario comparisons.
- Short event-log samples for validation and debugging.
- Confidence interval and paired-comparison tables.
- A concise final recommendation supported by statistics and logic.

Avoid excessive visual styling. The notebook should read like a polished Simulation course assignment.

---

## Security Rules

Never store secrets, API keys, tokens, or private credentials in frontend code.

Do not hardcode secrets in:

```text
JavaScript
React components
HTML
client-side environment variables
public config files
notebooks committed to source control
```

Use environment variables or backend-only configuration for sensitive values.

---

## Coding Agent Operating Principles

The coding agent should follow this order:

1. Read and understand this context.
2. Build a model map before writing code.
3. Define entities, resources, events, queues, and statistics.
4. Define sampling functions.
5. Define current-state event logic.
6. Implement minimal simulation skeleton.
7. Validate with small deterministic runs.
8. Add full stochastic behavior.
9. Add alternatives as configuration changes.
10. Add replication and statistical comparison.
11. Add tables, plots, and validation outputs only after simulation logic is coherent.

Do not jump directly to full code generation before the architecture is mapped.
