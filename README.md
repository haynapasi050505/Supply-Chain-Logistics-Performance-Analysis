# Supply Chain & Logistics Performance Intelligence

An end-to-end Power BI project that analyses delivery performance, cost efficiency and service quality across 20,000 shipments (Jan 2025 – Aug 2026), and turns the findings into operational recommendations.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Business Problem](#business-problem)
3. [Dataset](#dataset)
4. [Tools & Skills](#tools--skills)
5. [Data Preparation & Modelling](#data-preparation--modelling)
6. [KPIs & DAX Measures](#kpis--dax-measures)
7. [Dashboard Overview](#dashboard-overview)
8. [Key Insights](#key-insights)
9. [Recommendations](#recommendations)
10. [Project Structure](#project-structure)
11. [How to Use](#how-to-use)
12. [Limitations](#limitations)
13. [Author](#author)

---

## Project Overview

A logistics company ships products from multiple warehouses to cities across India using several shipping partners and modes. This project builds an interactive Power BI dashboard that helps operations and management teams monitor delivery reliability, understand the causes of delays, and control logistics costs.

## Business Problem

Management wants to answer:

- How reliable is our delivery performance overall, and where does it fall short?
- Which regions and shipping partners have the highest late-delivery rates?
- What are the main reasons for delays, and which are controllable?
- Which shipping modes drive logistics cost?
- How do demand peaks and seasonality affect operations?

## Dataset

A single flat table with **one row per shipment**.

| Item | Detail |
|---|---|
| Rows | 20,000 |
| Columns | 26 |
| Period | January 2025 – August 2026 |
| Currency | INR (₹) |
| Unique key | Shipment ID |
| Format | Excel (.xlsx) |

> **Note:** The dataset is synthetic. It was generated with Python (pandas, NumPy) to mimic realistic logistics patterns such as seasonality, partner reliability differences, regional delays and distance-based costs. It does not represent any real company.

### Data Dictionary

| Column | Description |
|---|---|
| Shipment ID | Unique shipment identifier (SHP-000001 to SHP-020000) |
| Order Date | Date the order was placed |
| Dispatch Date | Date the shipment left the warehouse (blank if cancelled) |
| Promised Delivery Date | Delivery date committed to the customer |
| Actual Delivery Date | Date the shipment was delivered (blank if cancelled) |
| Customer Segment | E-commerce, Retail, Wholesale, Corporate, Distributor |
| Product Category | Electronics, Apparel, FMCG, Pharmaceuticals, Home & Furniture, Automotive Parts, Industrial Equipment, Books & Stationery |
| Origin Warehouse | Warehouse city the shipment starts from |
| Destination City | Delivery city |
| Destination State | Delivery state |
| Destination Zone | North, South, East, West, Central, North East |
| Shipping Partner | Logistics carrier (e.g. BlueDart Express, Delhivery, DTDC, VRL Logistics) |
| Shipping Mode | Road, Rail, Air, Express |
| Distance (km) | Approximate route distance |
| Weight (kg) | Shipment weight |
| Units Shipped | Number of units in the shipment |
| Order Value (₹) | Value of goods shipped |
| Shipping Cost (₹) | Base freight charge |
| Fuel Surcharge (₹) | Fuel-related surcharge |
| Handling Cost (₹) | Warehouse and handling charge |
| Total Logistics Cost (₹) | Shipping + Fuel Surcharge + Handling |
| Delivery Status | Delivered On Time, Delivered Late, Returned, Cancelled |
| Delay Days | Actual minus Promised delivery date (negative = early, 0 = on time, positive = late) |
| Delay Reason | Cause of delay, or "None" |
| Damaged Flag | Yes / No |
| Customer Rating | 1–5 rating (blank where not given) |

**Handling of special cases:** cancelled orders have no dispatch/delivery date and `Delay Days` = 0. They should be excluded from delivery-performance KPIs. Customer Rating is intentionally blank for some shipments, as in real data.

## Tools & Skills

- **Power BI Desktop**: data modelling, DAX, visualisation
- **Power Query**: data cleaning and type setting
- **DAX**: KPIs, time intelligence, conditional logic
- **Python (pandas, NumPy)**: synthetic dataset generation
- **Excel**: source data

## Data Preparation & Modelling

1. Loaded the Excel file into Power Query and set correct data types (dates, decimals, whole numbers, text).
2. Verified 20,000 unique Shipment IDs and checked column quality.
3. Left genuine blanks (dispatch date, delivery date, rating) as null instead of filling them with zero.
4. Created a `Date` table with Year, Month, Month Year and Quarter, related to `Order Date`.
5. Kept `Shipments` as the single fact table and stored all measures in a dedicated Measures table.

## KPIs & DAX Measures

| KPI | Definition |
|---|---|
| Total Shipments | Count of all shipments |
| On-Time Delivery % | On-time deliveries ÷ delivered shipments |
| Late Delivery % | 1 − On-Time Delivery % |
| Total Logistics Cost | Sum of Total Logistics Cost (₹) |
| Average Logistics Cost | Total Logistics Cost ÷ Total Shipments |
| Average Customer Rating | Mean of Customer Rating |

```dax
Total Shipments = COUNTROWS(Shipments)

Delivered Shipments =
CALCULATE([Total Shipments],
    Shipments[Delivery Status] IN {"Delivered On Time","Delivered Late"})

On-Time Delivery % =
DIVIDE(
    CALCULATE([Total Shipments], Shipments[Delivery Status] = "Delivered On Time"),
    [Delivered Shipments])

Late Delivery % = 1 - [On-Time Delivery %]

Total Logistics Cost = SUM(Shipments[Total Logistics Cost (₹)])

Cost per Shipment = DIVIDE([Total Logistics Cost], [Total Shipments])

Cost per Kg = DIVIDE([Total Logistics Cost], SUM(Shipments[Weight (kg)]))

Avg Customer Rating = AVERAGE(Shipments[Customer Rating])
```

## Dashboard Overview

**Filters:** Destination Zone, Destination State, Shipping Partner

**KPI cards:** Total Shipments, On-Time Delivery %, Late Delivery %, Total Logistics Cost, Average Logistics Cost, Average Customer Rating

**Visuals:**

- Monthly Shipment Trend (line chart)
- Delivery Status Distribution (donut chart)
- Late Delivery % by Destination Zone (bar chart)
- Delay Reasons (bar chart)
- Logistics Cost by Shipping Mode (bar chart)
- Late Delivery Rate by Shipping Partner, North East (bar chart)

## Key Insights

### Headline performance
- 20,000 shipments with an **on-time delivery rate of 85.18%** and a late rate of **14.82%**, so roughly 1 in 7 delivered shipments arrives late.
- Total logistics cost is **₹31.75M**, an average of about **₹1.59K per shipment**.
- Average customer rating is **4.29**.
- Status split: 80.89% delivered on time, 14.07% delivered late, 3.18% returned, and the remainder cancelled.

### 1. The North East is the main problem region
Its late-delivery rate is **35.9%**, more than double that of every other zone (13.3%–16.1%). Central, North and West sit close together at around 13–14%, which shows the issue is regional rather than network-wide.

### 2. Partner choice drives North East performance
In the North East, VRL Logistics is late **68%** of the time, followed by Ecom Express (50%), DTDC (46.7%) and Gati (45.2%). BlueDart Express has the lowest late rate at **7.1%** and TCI Express is at 19.4%, a gap of about 60 percentage points between best and worst.

### 3. Most delays come from operational causes
Warehouse Processing Delay (591), Traffic Congestion (440) and Adverse Weather (433) are the top three delay reasons. Warehouse processing is the largest single cause and is directly controllable, unlike weather or traffic. Vehicle Breakdown, Partner Capacity Shortage and Festive Season Volume form the next group.

### 4. Strong festive-season peak
Shipment volume peaks in **October–November 2025** (about 1,531 shipments in the peak month, against roughly 800–1,000 in a typical month) and then returns to normal. *(August 2026 shows a drop because the data ends part-way through the month.)*

### 5. Air dominates logistics cost
Air accounts for about **₹15.6M, roughly half of total logistics cost**, followed by Road (₹9.8M), Rail (₹3.5M) and Express (₹2.9M). This is total cost, so it reflects both rate and volume. Cost per kg by mode should be compared before drawing conclusions on efficiency.

## Recommendations

1. **Re-allocate North East volume** away from the lowest-performing partners (VRL Logistics, Ecom Express, DTDC, Gati) towards more reliable carriers, or use them only for non-urgent loads.
2. **Set longer promise windows** and review routing for North East lanes to reduce missed commitments.
3. **Reduce warehouse processing delays** by targeting dispatch time, since this is the largest controllable cause.
4. **Plan capacity ahead of Q4** with buffer partner capacity and staffing, as festive volume and capacity shortages both appear in the delay reasons.
5. **Review Air usage** on short-distance or non-urgent lanes where Road or Rail could meet the delivery promise at lower cost.

## Project Structure

```
supply-chain-logistics-powerbi/
│
├── data/
│   └── Supply_Chain_Logistics_Dataset.xlsx
├── powerbi/
│   └── Supply_Chain_Logistics_Dashboard.pbix
├── screenshots/
│   └── dashboard.png
└── README.md
```



## Author

**Neha**
Data Analyst | Statistics (BSc & MSc), Savitribai Phule Pune University

- LinkedIn: [linkedin.com/in/neha-pasi](https://linkedin.com/in/neha-pasi)
- GitHub: [github.com/haynapasi050505](https://github.com/haynapasi050505)
