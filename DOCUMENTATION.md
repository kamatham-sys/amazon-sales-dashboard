# Project Documentation — Amazon Sales Dashboard

This document walks through the full data preparation and modeling process behind the dashboard, column by column, including the reasoning behind each decision.

---

## 1. Source Data

**File:** `Amazon_Sale_Report.csv`
**Rows:** ~129,000
**Original columns:** 24

The raw file is a single flat export of order-level transactions from an Amazon seller account, covering ~3 months (March 31 – June 29, 2022).

---

## 2. Modeling Approach: Star Schema

Rather than keeping one large flat table, the data was normalized into a **star schema**: one central fact table surrounded by dimension tables. This is standard practice in BI modeling because it:

- Reduces redundancy (repeated product/location attributes aren't duplicated across every order row)
- Improves performance in Power BI's data engine
- Makes relationships and filtering (slicers) more reliable
- Mirrors how real BI/data warehouse projects are structured

---

## 3. Column-by-Column Decisions

### Dropped entirely (no analytical value)

| Column | Reason |
|---|---|
| `Unnamed: 22` | Empty/junk column, artifact of the CSV export |
| `fulfilled-by` | ~70% blank (89,698 of ~129,000 rows); where filled, always "Easy Ship" — fully redundant with `Fulfilment` |
| `promotion-ids` | Comma-separated multi-value field requiring unpivoting; out of scope for this analysis |
| `Sales Channel` | 128,851 of 128,975 non-null rows = "Amazon.in" (99.9%); the remaining 124 "Non-Amazon" rows are too sparse to support meaningful analysis |
| `ship-country` | 128,942 of 128,975 non-null rows = "IN" (100% of known values); zero variation |
| `ship-city` | 9,459 unique postal codes but thousands of inconsistent city spellings/casings (e.g., "MUMBAI" / "Mumbai" / "Mumbai City"); no fuzzy-grouping tool was available in this Power BI Desktop setup, and city-level accuracy wasn't required since `ship-state` covers the geographic analysis need |

### Kept, with cleaning applied

| Column | Cleaning applied |
|---|---|
| `Date` | Converted from text (`MM-DD-YY`) to Date type using "Using Locale → English (United States)" to avoid the default locale misreading month/day order |
| `ship-state` | Trim → Clean → case standardization (UPPERCASE); manual correction of invalid entries found during profiling (e.g., "Nl" → "Nagaland", verified against the postal code range 797xxx) |
| `ship-postal-code` | Verified data type consistency (Whole Number) across Fact and Dimension tables before merging |
| `B2B` | Kept as a clean, complete boolean (no nulls) — useful for B2B vs B2C comparison |

### Kept as-is (already clean)

- `Order ID`, `SKU`, `Style`, `ASIN`, `Category`, `Size`, `Qty`, `Amount`, `currency`, `Status`, `Courier Status`, `Fulfilment`, `ship-service-level`

---

## 4. Table-by-Table Breakdown

### Fact_Amazon_Sales
One row per order line. Contains all numeric/transactional values plus foreign keys to each dimension:

- Order ID
- Date *(→ Dim_Date)*
- SKU *(→ Dim_Product)*
- Qty
- Amount
- currency
- Status *(kept at fact level — describes the state of the individual transaction, not a stable entity, so it doesn't belong in a dimension)*
- Courier Status *(same reasoning as Status — verified that a single SKU can have multiple different Courier Status values across different orders, confirming it's an event-level attribute, not a product attribute)*
- Location_ID *(→ Dim_Location, added via Merge Queries)*
- Fulfilment_ID *(→ Dim_Fulfilment, added via Merge Queries)*

### Dim_Product
**Key:** SKU (7,195 unique values)

Chosen over `ASIN` (7,190 unique, but 5 SKUs mapped to more than one ASIN and 10 ASINs mapped to more than one SKU — a small but real inconsistency) and over `Style` (only 1,377 unique values — too coarse, since one style spans multiple sizes/SKUs).

Columns: SKU, Style, Category, Size, ASIN
Deduplicated using `Table.Distinct` on SKU.

### Dim_Location
**Key:** Location_ID (surrogate, since no natural key existed)

Initial analysis showed `ship-postal-code` alone was unreliable as a key: 4,085 postal codes mapped to more than one city, and 213 postal codes even mapped to more than one state — due to inconsistent city/state text entry, not real-world duplication. `ship-city` was dropped for this reason (see above). The remaining columns (`ship-postal-code`, `ship-state`) were deduplicated together, then assigned a surrogate `Location_ID` via an Index column.

Columns: Location_ID, ship-postal-code, ship-state

### Dim_Fulfilment
**Key:** Fulfilment_ID (surrogate)

Built from three low-cardinality flags: `Fulfilment` (Merchant/Amazon), `ship-service-level` (Standard/Expedited), and `B2B` (True/False). Out of 8 mathematically possible combinations, only **5 combinations actually occur** in the data — deduplication correctly reduced the table to those 5 real rows before assigning a surrogate key.

Columns: Fulfilment_ID, Fulfilment, ship-service-level, B2B

### Dim_Date
Built as a standalone calendar table (not derived directly from the raw Date column) to support time-intelligence functions in DAX.

Columns: Date, Day, Month, Month Name, Quarter, Year, Day Name

---

## 5. Merge Queries — Connecting Fact to Dimensions

For dimensions using a surrogate key (`Dim_Location`, `Dim_Fulfilment`), the surrogate key had to be pulled into `Fact_Amazon_Sales` via **Merge Queries**, matching on the shared descriptive columns (e.g., `ship-postal-code` + `ship-state` for Location).

**Key troubleshooting note:** an initial merge attempt returned a high percentage of nulls in the joined key column. Root cause: `ship-state` had been cleaned (Trim/Case) in `Dim_Location` but not yet in `Fact_Amazon_Sales`, so values like `"Maharashtra"` (cleaned) and `"MAHARASHTRA"` (raw) failed to match exactly. Fix: applied identical cleaning steps to `ship-state` in `Fact_Amazon_Sales` before re-running the merge, which resolved the mismatch.

After a successful merge, the original raw `ship-state` / `ship-postal-code` columns were removed from `Fact_Amazon_Sales`, keeping only the surrogate key.

---

## 6. Lessons / Notes for Future Iterations

- Fuzzy grouping (for city-level cleanup) was not available in this Power BI Desktop environment; a manual Group-By-and-scan approach was used instead to catch the most obvious issues before deciding to drop the column entirely.
- Any future rework that reintroduces `ship-city` should budget significant time for manual/fuzzy deduplication given the scale of inconsistency (thousands of unique raw values).
- Merge operations across tables require exact-match values — always apply identical text cleaning to both sides of a merge key before joining.
