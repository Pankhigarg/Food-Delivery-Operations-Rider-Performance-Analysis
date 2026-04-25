# Food Delivery Operations & Rider Performance Analysis

> Analysed **42,383 food delivery orders** across 9,607 cities to identify operational bottlenecks, quantify the impact of traffic and festivals on delay, and build a composite rider performance leaderboard for 1,320 riders.

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

The output is a **5-page interactive Power BI dashboard** with a Python-driven analysis pipeline, a composite rider scoring model, and data-backed recommendations for ops leads.

---

## Executive Summary (Key Numbers)

| Metric | Value |
|---|---|
| Total Orders Analysed | **42,383** |
| Avg Delivery Time | **26.56 min** (SLA threshold: 30 min) |
| Avg Distance per Order | **9.78 km** |
| Overall Delay Rate | **0.31%** |
| Avg Rider Efficiency Score | **4.23** |
| Total Riders Scored | **1,320** |
| Cities Covered | **9,607** |
| Festival Impact Rate | **73.86%** of orders impacted |
| Peak Month Avg Delivery Time | **37.98 min** (festival season spike) |
| Festival Delay Premium | **+4.02 min** avg |
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

---

## Dashboard — 5 Pages

### Page 1 — Executive Overview
> High-level KPIs and operational pulse

- Total orders, avg delivery time, avg distance, delay rate %, avg efficiency
- **Delivery Time Distribution** — most orders fall in the 26–30 min bucket
- **Delay by City Tier** — Metropolitan: 21.63K on-time, Urban: 7.78K, Semi-Urban: 1.83K
- **Order Type Split** — Snack 25.2% · Buffet 25.1% · Drinks 24.9% · Meal 24.7%
- **Time by Vehicle Type** — Motorcycle fastest at 27.85 min avg; Electric scooter at 24.56 min

### Page 2 — Time Trends
> Monthly delivery time and volume analysis

- Peak month avg delivery time: **37.98 min** — driven by festival season
- Festival delay premium: **+4.02 min** (Festival: 45.50 min vs Non-festival: 37.45 min)
- **Delivery Time by Road Traffic** — Jam: 29.53 min · Medium: 26.80 · High: 26.64 · Low: 26.53
- Time trend line shows delivery time rising Feb → Mar then dropping Apr as festivals end
- Vehicle type time curve peaks in March and drops sharply in April

### Page 3 — Rider Leaderboard
> Composite performance scoring for 1,320 riders

- Avg Rider Score: **0.63** | Avg Efficiency: **0.71** | Avg Rating: **4.64**
- **Rider Category Distribution:**
  - Low Performer: **1,001 riders (75.83%)**
  - Average: **160 riders (12.12%)**
  - High Performer: **159 riders (12.05%)**
- **Avg Efficiency by Category:** Low Performer: 6.1 · Average: 4.0 · High Performer: 3.3
- Full leaderboard table with `Rider_ID`, rider score, delivery time, avg efficiency, avg rating, category

### Page 4 — Weather & Traffic
> External factor impact on delivery time

**Key Insights panel (auto-generated):**

| Factor | Delay Likelihood Multiplier |
|---|---|
| Festival = Yes | **3.42×** more likely to be delayed |
| Distance > 10.25 km | **2.61×** more likely to be delayed |
| Road Traffic = Jam | **2.45×** more likely to be delayed |

- **Worst Traffic Condition:** Jam
- **Worst Weather Condition:** Fog
- **Festival Impact %:** 73.86%
- Festival + Fog: 45.37 min avg · Festival + Cloudy: 44.93 min · Non-festival + Sunny: 21.88 min
- Motorcycle deliveries are **~1.39× more likely** to be impacted under high traffic

**Recommendations (dashboard-embedded):**
- Implement traffic-aware dynamic routing during peak congestion windows
- Increase rider allocation during festivals and bad weather
- Prioritise alternative vehicle types or route optimisation in high-density areas

### Page 5 — City & Geo Analysis
> Urban · Metropolitan · Semi-Urban breakdown with map

- Total cities: **9,607** | Best city avg time: **23.20 min**
- **Highest delay city type: Urban (19%)**
- Urban delay rate: ~81% of orders flagged as delayed in filtered view
- Avg time in Urban cities: 23.20 min · Avg distance: 9 km
- Interactive map plotting delay vs. on-time delivery locations globally

---

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
| Python (Pandas, NumPy, Matplotlib, Seaborn, Scikit-learn) | Data cleaning, feature engineering, scoring model, EDA |
| Power BI | 5-page interactive dashboard |
| GitHub | Version control, notebooks, CSV exports |

---

## Repository Structure

```
food-delivery-ops/
│
├── data/
│   ├── food_delivery_raw.csv              # Original dataset
│   └── food_delivery_processed.csv        # Cleaned + feature-engineered
│
├── notebooks/
│   ├── 01_eda.ipynb                       # Exploratory data analysis
│   ├── 02_feature_engineering.ipynb       # Rider efficiency score model
│   └── 03_root_cause_analysis.ipynb       # Delay driver analysis
│
├── powerbi/
│   └── delivery_ops_dashboard.pbix        # Full 5-page Power BI dashboard
│
├── outputs/
│   └── rider_leaderboard_scored.csv       # Final scored rider dataset
│
├── images/
│   ├── overview.png
│   ├── time_trends.png
│   ├── rider_leaderboard.png
│   ├── weather_traffic.png
│   └── city_geo.png
│
├── README.md
└── requirements.txt
```

---

## How to Run

```bash
# 1. Clone the repository
git clone https://github.com/yourusername/food-delivery-ops.git
cd food-delivery-ops

# 2. Install Python dependencies
pip install -r requirements.txt

# 3. Run notebooks in order
jupyter notebook notebooks/01_eda.ipynb

# 4. Open Power BI dashboard
# Load powerbi/delivery_ops_dashboard.pbix in Power BI Desktop
# Data source: outputs/rider_leaderboard_scored.csv
```

---

## Key Takeaways for Ops Teams

1. **Festivals are the single biggest delay driver** — 3.42× delay multiplier and 73.86% order coverage means festival planning is a critical ops lever
2. **Traffic jams compound with distance** — orders >10.25 km in jam conditions face a 2.45–2.61× combined delay risk
3. **75.83% of riders are low performers** — but the efficiency gap is context-sensitive; external adjustments reveal which riders genuinely underperform vs. those assigned tough routes
4. **Urban cities have the highest delay concentration** despite lower avg delivery times — density-driven congestion is the cause

---

## Skills Demonstrated

`Python` · `Pandas` · `Feature Engineering` · `Composite Scoring Model` · `Root Cause Analysis` · `Power BI` · `Geospatial Visualisation` · `Operational Analytics` · `Time Series Analysis` · `Data Visualisation`
