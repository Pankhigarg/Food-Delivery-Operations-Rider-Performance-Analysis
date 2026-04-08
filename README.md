# Food Delivery Operations & Rider Performance Analysis

![Badge](https://img.shields.io/badge/Project-PowerBI%20%7C%20Python-blue)

## Overview
This project analyzes 45,000+ food delivery orders to identify **key operational bottlenecks** and **factors affecting rider performance**.  
The goal is to understand the **external and internal drivers of delivery delays** and provide actionable recommendations to improve operational efficiency, similar to challenges faced at Swiggy/Instamart.

---

## Dataset
- Total Orders: **45,593**
- Columns include:
  - Rider details (`Delivery_person_ID`, `Delivery_person_Ratings`)
  - Order & pickup timestamps (`Time_Orderd`, `Time_Order_picked`)
  - Location (`Restaurant_latitude`, `Delivery_location_latitude`)
  - External factors (`Road_traffic_density`, `Weatherconditions`, `City`, `Festival`)
  - Internal factors (`Type_of_vehicle`, `Vehicle_condition`, `multiple_deliveries`, `Type_of_order`)
  - Metrics (`Time_taken(min)`, `distance_km`, `efficiency`, `delay_flag`)

**Data Sources:** Kaggle food delivery dataset & processed features.

---

## Project Goals
- Analyze **external factors** impacting delivery time: distance, traffic, city, weather, festivals  
- Analyze **internal factors** impacting rider performance: multiple deliveries, vehicle type/condition, rider efficiency  
- Compute **rider performance score** (efficiency + rating + order volume)  
- Categorize riders: High / Average / Low performers  
- Provide **actionable operational recommendations**

---

## Tech Stack
- **Python**: Pandas, NumPy, Matplotlib, Seaborn – Data cleaning, feature engineering, analysis  
- **Power BI**: Interactive dashboard to visualize external/internal factors and rider performance  
- **GitHub**: Repository for notebooks, CSV, and dashboards

---

## Key Features / Highlights
- **Rider Efficiency Metric**: min per km for fair comparison  
- **Composite Rider Score**: combines efficiency, rating, and total orders  
- **External vs Internal Analysis**: heatmaps, scatter plots, bar charts  
- **Dashboard-Ready CSV**: cleaned dataset with engineered metrics  
- **Actionable Insights**: training low performers, optimizing assignments, route management, fleet maintenance

---

## Insights
1. Distance, traffic, city, and festival periods are the **strongest external drivers** of delivery delays.  
2. Rider efficiency and operational decisions like **multiple deliveries** significantly affect performance.  
3. Rider rating is a weaker predictor of efficiency.  
4. High-performing riders handle more orders with lower time/km, whereas low performers have fewer orders and higher time/km.  
5. Targeted interventions can uplift low performers and improve overall delivery efficiency.

---

## Dashboard Layout

### Page 1: Overview
- KPI Cards: Avg Delivery Time, Avg Efficiency, % Delayed Orders, Total Orders  
- Rider Performance Pie Chart: High / Average / Low performers  
- Delivery Time Distribution Histogram

### Page 2: External Factors
- Scatter Plot: Distance vs Delivery Time  
- Bar Charts: Traffic vs Avg Delivery Time, Weather vs Avg Delivery Time, City vs Avg Delivery Time, Festival vs Avg Delivery Time  

### Page 3: Internal Factors
- Bar Charts: Multiple Deliveries, Vehicle Type, Vehicle Condition, Type of Order  
- Scatter Plot: Rider Rating vs Efficiency  

### Page 4: Combined / Root Cause
- Heatmap / Matrix: Traffic × Multiple Deliveries vs Avg Delivery Time  
- Rider Performance Table: Rider ID, Avg Efficiency, Avg Rating, Total Orders, Rider Category  
- Optional Trend Line: Avg Delivery Time over Order_Date  

---

## How to Use
1. Clone the repository:
```bash
git clone https://github.com/yourusername/food-delivery-ops.git
food-delivery-ops/
│
├── data/
│   └── food_delivery_powerbi.csv      # Cleaned CSV for Power BI
│
├── notebooks/
│   └── food_delivery_analysis.ipynb   # Python analysis and visualizations
│
├── powerbi/
│   └── dashboard.pbix                  # Power BI dashboard file
│
├── README.md                           # Project overview (this file)
└── images/                             # Screenshots for README or dashboard


Recommendations
Optimize rider assignments during peak hours and festival periods
Train low performers using high-performing rider benchmarks
Reduce multiple deliveries for low performers
Monitor KPI trends regularly via dashboards
Maintain vehicle fleet to support consistent delivery efficiency
Contact
Author: Pankhi Garg
