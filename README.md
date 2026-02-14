This revised **V2 README** is tailored to be the centerpiece of your student project documentation. It breaks down the mathematical weightings and the logical evolution from the V1 (Genoa vs. Cagliari) baseline to this advanced tactical engine.

---

# V2 Syndicate Engine: Tactical Match Simulation

## Project Overview

The **V2 Syndicate Engine** is an advanced sports analytics framework designed to forecast football match outcomes using **Monte Carlo simulations**, **non-linear tactical weighting**, and **temporal probability distributions**.

This engine represents a significant architectural shift from **V1** (the Genoa vs. Cagliari baseline). While V1 focused on static xG averages, **V2** introduces a "Tactical Friction" layer that accounts for team-specific setups, such as Defensive Line Depth and Sprint Speed differentials, to simulate how a game actually unfolds over 90 minutes.

---

## Logical Evolution: V1 to V2

| Feature | V1 (Baseline) | V2 (Syndicate) | Purpose |
| --- | --- | --- | --- |
| **xG Weighting** | Rolling 5-game average. | **Regression Triggering.** | Adjusts xG based on finishing efficiency (`bc_missed_L5`). |
| **Defense** | Simple goals-against average. | **Tactical Friction.** | Models the risk of a high defensive line against fast attackers. |
| **Fatigue** | Not modeled. | **Temporal Decay.** | Uses player availability to erode defensive resolve in H2. |
| **Props** | Linear averages. | **Poisson Trials.** | Simulates Corners and Cards as independent, random events. |

---

## Mathematical Architecture and Weighting

The V2 Engine relies on four primary mathematical pillars to determine match outcome probabilities.

### 1. Scoring Potency Index (SPI)

The base offensive threat is no longer just xG; it is adjusted for "Regression Triggering." We assume teams that have missed many "Big Chances" (`bc_missed`) are due for a finishing correction.

* **Weighting:** Every big chance missed provides a **4% boost** to the expected scoring potency, capped at **20%**, to account for mean reversion in finishing.

### 2. Tactical Friction and Exploitation

This formula determines how much one team's defensive style (Home) benefits the opponent's attacking pace (Away).

* **Logic:** A defensive line at **55m** is considered neutral. If the line is higher (e.g., 62m), the exploitation factor increases. This is then multiplied by the **Sprint Speed Delta**, assuming every **10 km/h** difference in speed doubles the effectiveness of the high-line exploitation.

### 3. Defensive Resolve and Leakage

V2 uses a non-linear divisor to prevent "infinite clean sheets." We calculate a resolve score that is then used to dampen the opponent's SPI.

* **Leakage Factor:** To maintain realism, a **0.12 leakage constant** is added to the final xG expectancy (). This ensures that even the strongest defense has a baseline probability of conceding in high-volatility environments.

### 4. Temporal Decay ()

This variable simulates the "late-game collapse" common in teams with poor squad depth or low player availability.

* **Impact:** The **Clutch Factor Index (CFI)** is squared to penalize teams that lack late-game composure. This multiplier is only applied to goals occurring in intervals 4, 5, and 6 (46–90 minutes).

---

## Data Input Requirements

To execute the V2 Engine, the following metrics must be provided for both teams:

| Category | Metric | Description |
| --- | --- | --- |
| **Attack** | `rolling_xg_L5` | Average Expected Goals over the last 5 matches. |
| **Tactical** | `High_Line_Depth` | Average height (meters) of the defensive line. |
| **Physical** | `Sprint_Speed_Avg` | Average top speed of the starting attacking unit. |
| **Reliability** | `CFI` | Clutch Factor Index (composure under pressure). |
| **Resources** | `Key_Player_Availability` | Percentage of the primary starting XI available for selection. |

---

## Execution Methodology

1. **Monte Carlo Simulation:** The engine runs **100,000 iterations** of the match.
2. **Interval Sampling:** For each iteration, the match is split into six 15-minute windows.
3. **Poisson Arrivals:** In each window, the number of goals is determined by , where  is the time-weighted  adjusted by the decay and resolve factors.
4. **Prop Generation:** Corners and Yellow Cards are generated as secondary Poisson trials based on **Field Tilt** (possession in the final third).

---

## Usage and Requirements

```python
# Libraries required
import pandas as pd
import numpy as np
import seaborn as sns
from scipy.stats import poisson

# Running the engine
results = run_v15_syndicate_engine(home_team_data, away_team_data)

```

---

**Would you like me to add a "Model Limitations" section that discusses the impact of un-modeled variables like weather or stadium noise?**
