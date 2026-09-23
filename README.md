Amazon Sales Dashboard (Power BI)

An end-to-end Power BI project analyzing Amazon marketplace sales data — from raw CSV to a cleaned star-schema data model and an interactive dashboard.

<img width="1321" height="738" alt="Screenshot 2026-09-24 000900" src="https://github.com/user-attachments/assets/0df738fb-dc84-455f-a4f6-a79a55d1d6b9" />


📊 Project Overview

This project takes a raw, real-world Amazon sales export (~129,000 order rows) and turns it into a clean, analysis-ready data model and dashboard. The focus was on proper dimensional modeling (star schema) rather than working off one flat table — including handling messy real-world issues like inconsistent city/state spellings, mixed data types, and duplicate records.

📁 Dataset
Source: Amazon Sale Report — Kaggle
Size: ~129,000 rows, 24 original columns
Period covered: March 31, 2022 – June 29, 2022
Content: Order-level sales data for an Indian D2C clothing seller on Amazon — includes order status, product details, shipping location, fulfilment type, and order value.
🛠️ Tools Used
Power BI Desktop — data modeling, DAX, dashboard visuals
Power Query (M) — data cleaning and transformation
DAX — measures for KPIs (Total Revenue, Total Orders, Average Order Value, etc.)
🧱 Data Model — Star Schema

Instead of using the raw data as a single flat table, the dataset was split into one fact table and four dimension tables:

Table	Type	Key	Description
Fact_Amazon_Sales	Fact	—	One row per order line: Order ID, Date, SKU, Qty, Amount, Status, Courier Status, B2B, and foreign keys to each dimension
Dim_Product	Dimension	SKU	Style, Category, Size, ASIN
Dim_Location	Dimension	Location_ID (surrogate)	ship-postal-code, ship-state
Dim_Fulfilment	Dimension	Fulfilment_ID (surrogate)	Fulfilment, ship-service-level, B2B
Dim_Date	Dimension	Date	Day, Month, Month Name, Quarter, Year, Day Name

Relationships: All dimension tables connect to Fact_Amazon_Sales in a 1-to-many relationship (one product/location/fulfilment/date → many order lines).

🧹 Data Cleaning Highlights

Real-world data is messy — here's what was addressed:

Locale-aware date parsing — raw dates were in MM-DD-YY text format; converted using Power Query's "Using Locale → English (United States)" to avoid misparsing.
Duplicate handling — deduplicated each dimension table on its key column before assigning surrogate keys, to avoid inflated or fragmented dimension rows.
Text standardization — applied Trim, Clean, and case standardization (UPPERCASE) to ship-state to fix inconsistent casing and stray characters (e.g., "Amravati." → "Amravati").
Typo correction — corrected invalid state values found during profiling (e.g., "Nl" → "Nagaland", confirmed via matching postal code range).
Dropped low-value columns — removed columns with no analytical value or excessive redundancy:
Unnamed: 22 — empty/junk export artifact
fulfilled-by — ~70% blank, redundant with Fulfilment
promotion-ids — messy multi-value field, out of scope for this analysis
Sales Channel — 99.9% single value ("Amazon.in"), no analytical variation
ship-country — 100% single value ("IN"), no analytical variation
ship-city — dropped after data-quality review; thousands of inconsistent spellings with low payoff versus using ship-state for geographic analysis
Cross-table consistency — ensured cleaned text formatting was applied identically in both Dim_Location and Fact_Amazon_Sales before merging, since Power Query merges require exact-match values.
📈 Dashboard

The dashboard (Overview page) includes:

KPI cards: Total Revenue, Total Quantity, Total Orders, Average Order Value
Order Status breakdown (pie chart)
Order flow (Shipped → Delivered/Cancelled/Returned) via a Sankey-style flow chart
Total Quantity by Category (bar chart)
Monthly Revenue Trend vs previous month (line chart)
Total Revenue by SKU (bar chart, top SKUs)
Slicers: Status, ship-state, Size, Month, Courier Status, Category, SKU
Key Metrics (from this dataset)
Total Revenue: ₹70.94M
Total Quantity: 109K units
Total Orders: 120K
Average Order Value: ₹589.43
📂 Repository Structure
├── Amazon_Sales_Dashboard.pbix     # Main Power BI file
├── data/
│   └── Amazon_Sale_Report.csv      # Raw source data
├── screenshots/
│   └── overview.png                # Dashboard screenshot(s)
├── README.md
└── DOCUMENTATION.md                # Detailed data modeling & cleaning steps
🚀 How to Use
Clone or download this repository
Open Amazon_Sales_Dashboard.pbix in Power BI Desktop
If prompted, update the data source path to point to data/Amazon_Sale_Report.csv on your machine
📖 Further Details

See DOCUMENTATION.md for a full breakdown of the Power Query transformation steps, column-by-column decisions, and modeling rationale.
