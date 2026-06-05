# Food Delivery Operations & Rider Performance Analysis

> Analysed **42,383 food delivery orders** across **9,607 delivery zones** to identify operational bottlenecks, quantify the impact of traffic and festivals on delay, and build a composite rider performance leaderboard for 1,320 riders

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=flat&logo=pandas&logoColor=white)
![Orders](https://img.shields.io/badge/Orders-42%2C383-brightgreen?style=flat)
![Riders](https://img.shields.io/badge/Riders-1%2C320-blue?style=flat)

---

## Overview

This project analyses a real-world food delivery dataset to answer the operational questions that matter to a logistics or quick-commerce team:

- **What external conditions cause the most delays?** (traffic, festivals, weather)
- **Which riders are underperforming — and why?**
- **How do delivery patterns vary across city tiers?**
- **Where should ops teams focus to reduce SLA breaches?**

The output is a 6-page interactive Power BI dashboard with a Python-driven analysis pipeline, a composite rider scoring model, and data-backed recommendations for ops leads
---
v2 note: This version corrects several measurement errors from v1 — including a broken delay rate calculation, a rider scoring inversion bug, and arbitrary category thresholds. All key metrics below reflect the corrected figures.

## Executive Summary (Key Numbers)

| Metric | Value |
|---|---|
| Total Orders Analysed | **42,383** |
| Avg Delivery Time | **26.56 min** (SLA threshold: 30 min) |
| Avg Distance per Order | **9.78 km** |
| Overall Delay Rate | **30.37%** |   (Fixed from v1's incorrect 0.31%)
| Avg Rider Efficiency Score | **4.23** |
| Total Riders Scored | **1,320** |
| Cities Covered | **9,607** |
| Festival Impact Rate | **73.86%** of orders impacted |
| Peak Month Avg Delivery Time | **45.50 min**  (festival season spike) |
| Festival Delay Premium |**+19.33 min**| (45.50 min vs 26.17 min non-festival)
| Avg Pickup Wait | **9.98 min** |


---

## Dataset

**Total Orders:** 42,383 | **Date Range:** Feb 11, 2022 – Apr 6, 2022

| Feature Category | Columns |
|---|---|
| Rider details | `Delivery_person_ID`, `Delivery_person_Ratings` |
| Dispatch & timing | `Time_Orderd`, `Time_Order_picked`, `Time_taken(min)` |
| Location | `Restaurant_latitude/longitude`, `Delivery_latitude/longitude` |
| External factors | `Road_traffic_density`, `Weatherconditions`, `City`, `Festival` |
| Internal factors | `multiple_deliveries`, `Vehicle_condition`, `Type_of_vehicle`, `Type_of_order` |

Clarification: The raw City column contains 9,607 unique values representing delivery zones, not distinct cities. Analysis uses the city_tier grouping (Metropolitan / Urban / Semi-Urban) for all geographic comparisons.
---
## v2 Fixes & Improvements
 
### Bug 1 — Delay Rate Was Wrong (0.31% → 30.37%)
 
v1 applied `delay_flag` inconsistently, producing an implausibly low 0.31% delay rate. v2 applies a clean SLA threshold:
 
```python
SLA_THRESHOLD = 30  # minutes
df['delay_flag'] = np.where(df['Time_taken(min)'] > SLA_THRESHOLD, 'Delay', 'On Time')
# Result: 30.37% of orders delayed — consistent with Power BI dashboard
```
 
### Bug 2 — Rider Scoring Was Inverted
 
In v1 I normalised efficiency (min/km) but forgot to invert it due to which  meaning slower riders scored higher. v2 explicitly inverts both efficiency and delay rate so that lower raw values produce higher normalised scores:
 
```python
rider_agg['eff_norm']   = 1 - scaler.fit_transform(rider_agg[['avg_min_per_km']])  # lower = faster = better
rider_agg['delay_norm'] = 1 - scaler.fit_transform(rider_agg[['delay_rate']])       # lower = fewer delays = better
```
 
 

 
### New Feature — Multi-Drop Timing Adjustment
 
v1 did not account for multiple simultaneous deliveries, unfairly penalising riders assigned multi-drop routes. v2 adjusts delivery time before scoring:
 
```python
# riders with 3 simultaneous drops will naturally take longer — factor it out
df['adj_time'] = df['Time_taken(min)'] / (1 + 0.2 * df['multiple_deliveries'])
df['adj_min_per_km'] = df['adj_time'] / df['distance_km_safe']
```
 
### New Feature — Route Difficulty Index
 
v2 builds a difficulty index per delivery from traffic, weather, and festival conditions, incorporated as a fairness adjustment in composite scoring:
 
```python
traffic_map  = {'Low': 0, 'Medium': 1, 'High': 2, 'Jam': 3}
weather_map  = {'Sunny': 0, 'Cloudy': 1, 'Windy': 1, 'Sandstorms': 2, 'Stormy': 2, 'Fog': 2}
festival_map = {'No': 0, 'Yes': 1}
 
df['difficulty_index'] = (
    df['traffic_score'] + df['weather_score'] + df['festival_score']
) / 5.0  # normalised 0–1
```
 
### New Feature — Logistic Regression Delay Model
 
v2 adds a logistic regression model to quantify the probability of delay given external conditions, producing interpretable multipliers for the Power BI Key Influencers visual:
 
| Factor | Delay Likelihood Multiplier |
|---|---|
| Festival = Yes | **3.42×** |
| Distance > 10.25 km | **2.61×** |
| Road Traffic = Jam | **2.45×** |
| Vehicle condition = 0 | **1.71×** |
| Weather = Cloudy or Fog | **1.65×** |
| Vehicle type = Motorcycle | **1.39×** |
 
### New Feature — Vehicle Fuel Type Classification
 
v2 classifies vehicle sub-types into IC Engine vs EV, enabling fleet-level performance comparisons backed by a Mann-Whitney U significance test:
 
```python
vehicle_fuel_map = {
    'motorcycle'      : 'IC Engine',
    'scooter'         : 'IC Engine',
    'electric_scooter': 'EV',
}
df['vehicle_fuel_type'] = df['Type_of_vehicle'].map(vehicle_fuel_map)
# Result: EV averages 24.52 min vs IC Engine 26.61 min — statistically significant
```
 
### New Feature — Time-of-Day Segmentation
 
v2 derives order hour from timestamps and bins into six named periods used in the heatmap and trend charts:
 
```python
df['time_of_day'] = pd.cut(
    df['order_hour'],
    bins   = [0, 6, 11, 14, 17, 20, 24],
    labels = ['Late night', 'Breakfast', 'Lunch', 'Afternoon', 'Dinner', 'Night'],
    right  = False
)
```
 
---
 
## Rider Scoring Model (v2)
 
A four-step composite scoring model that ranks all 1,320 riders fairly, adjusting for route difficulty and multi-drop assignments.
 
### Step 1 — Raw Metrics
 
Aggregate per-rider: avg delivery time, avg min/km, avg rating, delay rate, avg difficulty.
 
### Step 2 — Multi-Drop Adjustment
 
`adj_time = raw_time ÷ (1 + 0.2 × simultaneous_orders)` — makes comparisons fair across single and multi-drop assignments.
 
### Step 3 — Min-Max Normalisation
 
All components scaled 0–1 across the 1,319 riders with ≥5 orders. Efficiency and delay rate are inverted so lower raw values → higher scores.
 
### Step 4 — Weighted Blend
 
```python
composite_score = (
    0.35 * eff_norm       +   # delivery efficiency — strongest operational signal
    0.30 * delay_norm     +   # delay rate — direct SLA impact
    0.25 * rating_norm    +   # customer rating — service quality indicator
    0.10 * difficulty_norm    # route difficulty bonus — fairness adjustment
)
```
 
**Weight rationale:** Efficiency and delay rate were weighted highest because they showed the strongest correlation with operational outcomes in the EDA. Customer rating was retained but weighted lower — ratings varied by only 0.09 across rider tiers, making them a weak discriminator. Difficulty (10%) acts as a contextual fairness adjustment rather than a primary performance signal.
 
**Output categories (percentile-based):**
 
| Category | Score Range | Rider Count | Share |
|---|---|---|---|
| High Performer | Top 33% | 435 | 33% |
| Average | Middle 34% | 447 | 34% |
| Low Performer | Bottom 33% | 437 | 33% |


 
## Dashboard — 6 Pages

### Page 1 — Executive Overview
> High-level KPIs and operational pulse

- Total orders, avg delivery time, avg distance, delay rate %, avg efficiency
- **Delivery Time Distribution** — most orders fall in the 26–30 min bucket
- **Delay by City Tier** — Metropolitan: 21.63K on-time, Urban: 7.78K, Semi-Urban: 1.83K
- **Order Volume by Time of Day** — Night 41.78% · Dinner 30.73% · Lunch 8.15%
- **Time by Vehicle Type** — IC Engine: 26.61 min avg; EV: 24.52 min
<img width="1440" height="758" alt="image" src="https://github.com/user-attachments/assets/9c154b44-d6f8-4a60-ad19-3b6992acf218" />

### Page 2 — Time Trends
> Monthly delivery time and volume analysis

- Peak month avg delivery time: **45.50 min** — festival season spike
- Festival delay premium: **+19.33 min** (Festival: 45.50 min vs Non-festival: 26.17 min)
- **Delivery Time by Road Traffic** — Jam: 21.67 min transit · Low: 11.70 min transit
- **Time-Traffic Heatmap** — Dinner + Jam peaks at 31.2 min; Breakfast + Low at 19.5 min

### Page 3 — Rider Leaderboard
> Composite performance scoring for 1,320 riders

-- Avg Rider Score: **0.60** | Avg Min/Km: **3.52** | High Performers: **435** | Low Performers: **437**
- **Rider Category Distribution** — ~33% each across High / Average / Low (corrected from v1)
- **Avg Min/Km by Category** — High Performer: 2.8 · Average: 3.1 · Low Performer: 4.6
- Full leaderboard table with composite score, avg min/km, avg rating, total orders, delay rate, and rank


### Page 4 — Score Methodology
How each rider's composite score is calculated
 
- Step-by-step formula breakdown with weight rationale
- Multi-drop timing adjustment explanation
- Performance category thresholds and rider counts
- Average time by external vs internal factors (city tier highest at 49.7 min)
- 
### Page 5 — Weather & Traffic
> External factor impact on delivery time

**Key Insights panel (auto-generated):**

| Factor | Delay Likelihood Multiplier |
|---|---|
| Festival = Yes | **3.42×** more likely to be delayed |
| Distance > 10.25 km | **2.61×** more likely to be delayed |
| Road Traffic = Jam | **2.45×** more likely to be delayed |
- **Key Influencers visual** — powered by logistic regression, showing delay multipliers per factor
- **Worst Traffic Condition:** Jam | **Worst Weather:** Fog | **Festival Impact:** 74.24%
- Festival + adverse weather: 73% increase in delivery time
- Motorcycle deliveries 1.39× more likely to be impacted under high traffic
- **Time-of-Day × Traffic heatmap** — Dinner + Jam at 31.2 min is the worst operational window
- Festival + Fog: 45.37 min avg · Festival + Cloudy: 44.93 min · Non-festival + Sunny: 21.88 min

**Recommendations (dashboard-embedded):**
- Implement traffic-aware dynamic routing during peak congestion windows
- Increase rider allocation during festivals and bad weather
- Prioritise alternative vehicle types or route optimisation in high-density areas


### Page 6 — City & Geo Analysis
Urban · Metropolitan · Semi-Urban breakdown with map
 
- Total delivery zones: **9,607** | Best city tier avg time: **23.20 min** (Urban)
- **Delay Rate by Tier** — Semi-Urban: 100% · Urban: 81.19% · Metropolitan: 33.48%
- Semi-Urban avg time: 49.71 min over just 13 km — coverage gap, not distance
- Interactive map plotting delay vs on-time delivery locations across India

## Rider Scoring Model

A composite scoring model built in Python to rank all 1,320 riders fairly, adjusting for route difficulty and external conditions:

```python
# Composite score components
Rider_Score = f(
    avg_delivery_time,        # normalised per km
    avg_efficiency,           # orders per unit time adjusted for distance
    avg_rating,               # customer-facing rating
    external_adjustments      # traffic + festival penalty factored out
)

# Output categories
Category = "High Performer"  if score >= threshold_high
         = "Average"         if score >= threshold_avg
         = "Low Performer"   otherwise
```

**Output:** Dashboard-ready CSV with all 1,320 riders scored, categorised, and ready for fleet manager review.

---


## Tech Stack
 
| Tool | Purpose |
|---|---|
| Python (Pandas, NumPy, Matplotlib, Seaborn, Scikit-learn, SciPy) | Data cleaning, GPS validation, feature engineering, scoring model, logistic regression, EDA |
| Power BI | 6-page interactive dashboard with Key Influencers AI visual |
| DAX | Custom measures for delay rate, composite score, festival premium |
| Star Schema | Fact + dimension tables for scalable Power BI modelling |
| GitHub | Version control, notebooks, CSV exports |
 
---

## Repository Structure
 
```
food-delivery-ops/
│
├── data/
│   ├── raw_dataset.csv                    # Original dataset
│   └── food_delivery_processed.csv        # Cleaned + feature-engineered (v2 output)
│
├── notebooks/
│   ├── v1_original.ipynb                  # Original EDA and scoring (reference only)
│   └── v2_improved.ipynb                  # Corrected pipeline: cleaning, GPS validation,
│                                          #   feature engineering, scoring, logistic regression
│
├── powerbi/
│   └── delivery_ops_dashboard.pbix        # Full 6-page Power BI dashboard
│
├── outputs/
│   └── rider_leaderboard_scored.csv       # Final scored rider dataset (1,320 riders)
│
├── images/
│   ├── overview.png
│   ├── time_trends.png
│   ├── rider_leaderboard.png
│   ├── score_methodology.png
│   ├── weather_traffic.png
│   └── city_geo.png
│
├── README.md
└── requirements.t

---

## How to Run
 
```bash
# 1. Clone the repository
git clone https://github.com/yourusername/food-delivery-ops.git
cd food-delivery-ops
 
# 2. Install Python dependencies
pip install -r requirements.txt
 
# 3. Run the v2 notebook (single file — full pipeline)
jupyter notebook notebooks/v2_improved.ipynb
 
# 4. Open Power BI dashboard
# Load powerbi/delivery_ops_dashboard.pbix in Power BI Desktop
# Data source: outputs/rider_leaderboard_scored.csv + outputs/food_delivery_processed.csv
```
 
---
 
## Key Takeaways for Ops Teams
 
1. **Delays happen in transit, not at pickup.** Pickup wait time varies by only ~4 seconds across all traffic levels. Once a rider is moving, Jam conditions add 88% more time than Low traffic — dynamic routing during peak hours is the highest-leverage intervention.
2. **Festivals are the single biggest delay driver.** A 3.42× delay multiplier and 99% festival delay rate means surge staffing must be pre-planned, not reactive.
3. **Semi-urban zones need network investment, not routing fixes.** 100% delay rate over short distances (avg 13 km) signals a coverage gap — more riders are needed, not smarter routes.
4. **EV scooters are ~3 minutes faster per delivery** — statistically significant (Mann-Whitney U). Fleet mix optimisation toward EVs is a measurable efficiency lever.
5. **Customer ratings are a weak performance signal.** Min/km varies by 66% across rider tiers; ratings vary by only 0.09. Ops decisions on coaching and assignment should be driven by efficiency and delay rate, not ratings.

---
 
## Skills Demonstrated
 
`Python` · `Pandas` · `NumPy` · `Scikit-learn` · `SciPy` · `GPS Validation` · `Haversine Distance` · `Feature Engineering` · `Composite Scoring Model` · `Logistic Regression` · `Root Cause Analysis` · `Power BI` · `DAX` · `Star Schema` · `Geospatial Visualisation` · `Operational Analytics` · `Time Series Analysis` · `Data Visualisation`
