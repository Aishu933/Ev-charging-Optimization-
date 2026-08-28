# EV Charging and Journey Time Optimisation

Dissertation code for analysing electric vehicle charging behaviour and
simulating a 600-mile journey under different driving speeds and charging
strategies.

The project has two halves:

1. **Empirical analysis** of a dataset of 3,500 EV charging sessions
   (duration, energy delivered, charger power, pricing, grid load).
2. **Journey simulation** of a 600-mile trip, testing seven charging
   strategies across speeds from 30 to 80 mph to find the combination that
   minimises total journey time.

---

## Contents

| File | Description |
| --- | --- |
| `Dissertation_Final_code.ipynb` | Main notebook — analysis, simulations, and plots |
| `ev_charging_sessions.csv` | Input dataset |

---

## Dataset

The notebook expects a file named `ev_charging_sessions.csv` in the working
directory. It contains 3,500 charging sessions with 11 columns:

| Column | Type | Description |
| --- | --- | --- |
| `date` | date | Date of the charging session |
| `start_hour` | int | Hour the session started (0–23) |
| `city` | str | City of the charging station |
| `station_id` | str | Charging station identifier |
| `charger_kw` | int | Rated charger power (kW) |
| `session_minutes` | int | Session duration in minutes |
| `energy_kwh` | float | Energy delivered (kWh) |
| `price_per_kwh` | float | Unit price of electricity |
| `session_cost` | float | Total cost of the session |
| `grid_load_mw` | float | Grid load at the time of charging (MW) |
| `peak_hour` | int | 1 if the session occurred during peak hours, 0 otherwise |

There are no missing values.

---

## Requirements

- Python 3.8 or later
- pandas
- numpy
- matplotlib
- seaborn
- scikit-learn

Install with:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn
```

---

## Running the notebook

The notebook was written in Google Colab but runs anywhere Jupyter runs.

**Locally:**

```bash
jupyter notebook Dissertation_Final_code.ipynb
```

Place `ev_charging_sessions.csv` in the same folder as the notebook, then run
the cells in order from top to bottom. Later sections depend on variables
defined earlier, so the notebook will not run correctly out of sequence.

**In Google Colab:** upload both the notebook and the CSV, then run all cells.

Random simulations use `random.seed(42)`, so results are reproducible.

---

## Notebook structure

### Part 1 — Exploratory data analysis (sections 1–13)

- Loading the data, checking for missing values, and deriving `year`, `month`,
  and `day_of_week` from the date
- Deriving a `charging_rate` variable (kWh delivered per hour of session time)
- Distributions of session duration and energy delivered
- Charger power vs energy delivered, and average session time by charger power
- Peak vs off-peak charging duration comparison
- Correlation matrix across the numeric variables
- Range-based metrics: `miles_added` (assuming 4 miles/kWh) and
  `minutes_per_mile`, linking charging time to journey range gained
- A linear regression predicting `session_minutes` from charger power, energy
  delivered, grid load, and peak-hour flag (**R² = 0.17** on the test set)

### Part 2 — Battery and efficiency models

- Battery discharge over a 600-mile journey
- Charging curve showing the slowdown above 80% state of charge
- Charging time vs energy added
- Speed vs efficiency, energy consumption, and effective range, using the
  model `consumption = 0.15 + 0.00004 × speed²` kWh/mile

### Part 3 — 600-mile journey scenarios

All scenarios assume a 75 kWh battery and a 50 kW average charging rate.

| Scenario | Strategy |
| --- | --- |
| 1 | 300 miles at 50 mph, charge to 100%, then a further 300 miles |
| 2 | Three 200-mile legs at 50 mph with charges to 80% in between |
| 3 | Drive to 0%, charge to 100%, repeat — tested at 30–80 mph |
| 4 | Drive to 20%, charge to 100%, repeat — tested at 30–80 mph |
| 5 | Drive to 0%, charge to 80%, repeat — tested at 30–80 mph |
| 6 | Drive to 20%, charge to 80%, repeat — tested at 30–80 mph |
| 7 | Randomised: drive to a random 0–50% state of charge, then recharge to a random 50–100%, repeated over 200 trials per speed |

The final two cells extend the random approach: one sweeps every speed from
30 to 80 mph with a linear efficiency model, and the last runs 10 trials at
80 mph using a random charging floor below 10% with a final top-up sized to
reach the destination exactly.

Each scenario produces a per-speed comparison table (charging stops, driving
time, charging time, total journey time) and a comparison graph.

---

## Key results

Total journey time for 600 miles, by scenario and speed (hours):

| Speed | Sc. 3 (0→100%) | Sc. 4 (20→100%) | Sc. 5 (0→80%) | Sc. 6 (20→80%) | Sc. 7 (random, best) |
| --- | --- | --- | --- | --- | --- |
| 30 mph | 21.50 | 21.20 | 21.20 | 21.80 | 20.71 |
| 40 mph | 16.50 | 16.20 | 16.20 | 16.80 | 16.00 |
| 50 mph | 13.50 | 14.40 | 13.20 | 13.80 | 13.20 |
| 60 mph | 11.50 | 12.40 | 12.40 | 11.80 | 11.55 |
| 70 mph | 11.57 | 10.97 | 10.97 | 11.27 | 10.52 |
| 80 mph | 10.50 | 11.10 | 11.10 | 11.10 | 10.09 |

The fastest journey found across all scenarios is **10.085 hours at 80 mph**
under the randomised strategy, using four charging stops. Driving faster
lowers efficiency and adds charging stops, but the time saved on the road
still outweighs the extra charging time over 600 miles.

The regression on the observed session data explains only about 17% of the
variance in session duration, indicating that charger power, energy delivered,
grid load, and peak-hour status alone do not determine how long a session
lasts.

---

## Assumptions and limitations

- Efficiency values by speed (5.5 miles/kWh at 30 mph down to 3.0 at 80 mph)
  are assumed, chosen to represent typical EV behaviour rather than measured
  from the dataset.
- Charging is modelled at a constant 50 kW. Real charging tapers sharply above
  roughly 80% state of charge; the charging-curve section illustrates this but
  the journey simulations do not apply it.
- Journeys assume flat terrain, no traffic, no weather effects, no battery
  temperature or degradation effects, and immediate charger availability with
  no queuing.
- Detour time to reach a charger and any non-charging stops are excluded.
- The 4 miles/kWh figure used for `miles_added` in Part 1 is a separate flat
  assumption from the speed-dependent efficiency used in Parts 2 and 3.
