# Amazon Sales Dashboard — Power BI

An end-to-end **Power BI sales analytics project** built using Amazon marketplace data. The project covers the complete workflow from **raw CSV data → Power Query transformation → star-schema data model → DAX measures → interactive dashboard**.

![Amazon Sales Dashboard](https://github.com/user-attachments/assets/0df738fb-dc84-455f-a4f6-a79a55d1d6b9)

---

## 📊 Project Overview

This project analyzes approximately **129,000 Amazon sales records** from an Indian D2C clothing seller.

Instead of building the report directly on the raw dataset, the data was transformed into a **clean star-schema model** with a dedicated fact table and dimension tables.

The project focuses on:

* Data cleaning and transformation
* Dimensional data modeling
* Surrogate key creation
* Fact and dimension table design
* DAX-based KPI calculations
* Interactive Power BI visualization
* Handling real-world data-quality issues

---

## 📁 Dataset

**Source:** Amazon Sale Report — Kaggle

| Attribute        | Details                        |
| ---------------- | ------------------------------ |
| Rows             | ~129,000                       |
| Original Columns | 24                             |
| Period           | March 31, 2022 – June 29, 2022 |
| Data Type        | Order-level sales data         |
| Market           | Amazon India                   |
| Business         | D2C clothing seller            |

The dataset contains information about orders, products, quantities, sales amounts, order status, fulfilment, courier status, shipping location, and customer/order attributes.

---

## 🛠️ Tools & Technologies

* **Power BI Desktop** — Data modeling, DAX, and dashboard development
* **Power Query (M)** — Data cleaning and transformation
* **DAX** — KPI and analytical measures
* **CSV** — Raw data source

---

## 🧱 Data Model — Star Schema

The raw dataset was transformed into a **star-schema model** consisting of one fact table and four dimension tables.

| Table               | Type      | Key           | Purpose                                                    |
| ------------------- | --------- | ------------- | ---------------------------------------------------------- |
| `Fact_Amazon_Sales` | Fact      | Foreign Keys  | Stores order-level sales transactions                      |
| `Dim_Product`       | Dimension | SKU           | Product attributes such as Style, Category, Size, and ASIN |
| `Dim_Location`      | Dimension | Location_ID   | Shipping postal code and state information                 |
| `Dim_Fulfilment`    | Dimension | Fulfilment_ID | Fulfilment, service level, and B2B attributes              |
| `Dim_Date`          | Dimension | Date          | Day, Month, Quarter, Year, and Day Name                    |

### Fact Table

`Fact_Amazon_Sales` contains the transactional data, including:

* Order ID
* Date
* SKU
* Quantity
* Amount
* Order Status
* Courier Status
* B2B
* Foreign keys to dimension tables

### Dimension Tables

**Dim_Product**

* SKU
* Style
* Category
* Size
* ASIN

**Dim_Location**

* Location_ID
* Shipping Postal Code
* Shipping State

**Dim_Fulfilment**

* Fulfilment_ID
* Fulfilment
* Shipping Service Level
* B2B

**Dim_Date**

* Date
* Day
* Month
* Month Name
* Quarter
* Year
* Day Name

### Relationships

All dimension tables have a **1-to-many relationship** with `Fact_Amazon_Sales`.

```text
                 Dim_Product
                     │
                     │ 1 : *
                     ▼
Dim_Location ──► Fact_Amazon_Sales ◄── Dim_Fulfilment
                     ▲
                     │ 1 : *
                     │
                  Dim_Date
```

This structure improves data organization, filtering, and scalability compared with using a single flat table.

---

## 🧹 Data Cleaning & Transformation

Several real-world data-quality issues were identified and addressed using Power Query.

### Date Parsing

The original date column contained text values in **MM-DD-YY format**.

The column was converted using:

**Using Locale → English (United States)**

This ensured that dates were interpreted correctly rather than being incorrectly parsed according to the system locale.

### Duplicate Handling

Dimension tables were deduplicated based on their respective business keys before surrogate keys were assigned.

This prevented duplicate dimension records and helped maintain consistent relationships with the fact table.

### Text Standardization

Text fields were cleaned using:

* Trim
* Clean
* Case standardization
* Value replacement

For example, inconsistent state formatting and stray characters were standardized before creating relationships.

### Data Quality Corrections

Invalid or inconsistent values identified during data profiling were corrected where they could be reliably validated.

Example:

`Nl` → `Nagaland`

The correction was validated using the associated postal-code information.

### Column Reduction

Columns with limited analytical value or excessive redundancy were removed:

| Column          | Reason                                                                   |
| --------------- | ------------------------------------------------------------------------ |
| `Unnamed: 22`   | Empty/junk export column                                                 |
| `fulfilled-by`  | ~70% blank and redundant with Fulfilment                                 |
| `promotion-ids` | Complex multi-value field; outside project scope                         |
| `Sales Channel` | ~99.9% single value (`Amazon.in`)                                        |
| `ship-country`  | 100% single value (`IN`)                                                 |
| `ship-city`     | High spelling inconsistency; limited analytical value for this dashboard |

### Cross-Table Consistency

The same text-cleaning rules were applied to relevant columns in both the fact and dimension tables.

This was particularly important before Power Query merges because merge operations require **exact matching values**.

---

## 📈 Dashboard

The **Overview** page provides an interactive summary of Amazon sales performance.

### KPI Cards

* **Total Revenue:** ₹70.94M
* **Total Quantity:** 109K units
* **Total Orders:** 120K
* **Average Order Value:** ₹589.43

### Visualizations

* **Order Status Breakdown** — Pie chart
* **Order Flow** — Sankey-style flow visualization
* **Quantity by Category** — Bar chart
* **Monthly Revenue Trend** — Line chart with previous-month comparison
* **Revenue by SKU** — Bar chart showing top-performing SKUs

### Interactive Filters

Users can filter the dashboard by:

* Status
* Shipping State
* Size
* Month
* Courier Status
* Category
* SKU

---

## 📂 Repository Structure

```text
Amazon-Sales-Dashboard/
│
├── Amazon_Sales_Dashboard.pbix
│
├── data/
│   └── Amazon_Sale_Report.csv
│
├── screenshots/
│   └── overview.png
│
├── README.md
│
└── DOCUMENTATION.md
```

### File Description

* `Amazon_Sales_Dashboard.pbix` — Main Power BI report
* `Amazon_Sale_Report.csv` — Raw source dataset
* `screenshots/` — Dashboard screenshots
* `DOCUMENTATION.md` — Detailed cleaning, transformation, and modeling documentation

---

## 🚀 How to Use

1. Clone or download this repository.
2. Open `Amazon_Sales_Dashboard.pbix` using **Power BI Desktop**.
3. If prompted, update the data source path.
4. Point the data source to:

```text
data/Amazon_Sale_Report.csv
```

5. Refresh the dataset if required.
6. Explore the dashboard using the available slicers and visualizations.

---

## 📖 Documentation

For a detailed explanation of the project, including:

* Power Query transformations
* Data-cleaning decisions
* Column-level changes
* Star-schema design
* Surrogate key creation
* Relationships
* Modeling rationale

see **[DOCUMENTATION.md](DOCUMENTATION.md)**.

---

## 🎯 Key Takeaways

This project demonstrates an end-to-end Power BI workflow involving:

**Raw Data → Data Cleaning → Data Modeling → DAX → Visualization → Interactive Dashboard**

The main emphasis is on building a **structured, analysis-ready data model** rather than simply creating visuals from a single raw table.
