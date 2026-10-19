# E-Commerce Sales Dashboard (Power BI)

An end-to-end Power BI project: raw data cleaning in Power Query, a star-schema data model with a calendar table, DAX measures, and a 3-page interactive dashboard analysing an online store in Azerbaijan (2024-2025).

> **Note:** the dataset is **synthetic**. It was generated for portfolio practice and deliberately contains typical real-world data quality problems (duplicates, mixed date formats, typos, impossible values, missing values).

## Dashboard preview

| Executive Overview | Product & Customer | Operations & Quality |
|---|---|---|
| ![Executive Overview](screenshots/01_executive_overview.png) | ![Product & Customer](screenshots/02_product_customer.png) | ![Operations & Quality](screenshots/03_operations_quality.png) |

## Business questions

1. **Executive Overview:** How is revenue trending? Which categories, cities and channels drive it?
2. **Product & Customer:** Which products sell best? Who are the customers (age, gender) and how do they pay?
3. **Operations & Quality:** Are we meeting delivery targets? Where do returns and cancellations hurt revenue? How does satisfaction change over time?

## Key findings (2025)

- Net revenue of **598,378 AZN** from **1,507 orders**, **+1.3%** vs. 2024 (590,679 AZN).
- **Electronics** generates roughly half of all net revenue (297K of 598K AZN).
- **Baku** is the largest city by revenue and also the fastest to deliver: about 2.7 days on average vs. about 4.6 days elsewhere.
- Both years peak in **November**, driven by seasonal demand.
- Return rate is **7.8%** and cancellation rate **8.3%**. The highest return rate is in **Electronics via Marketplace (14.1%)**.
- Average delivery time is **3.8 days** against a 4-day target; average rating is **3.9 / 5**.

## Tools & skills

- **Power Query (M):** data cleaning and transformation
- **Power BI Desktop:** data modelling, report design, custom theme (JSON)
- **DAX:** measures, time intelligence (year-over-year)
- **Visuals:** KPI cards, line, bar, column, donut, treemap, gauge, waterfall, heatmap, combo chart, slicers with cross-page sync

> The Operations & Quality page uses custom visuals from Microsoft AppSource (waterfall, gauge, heatmap). They are embedded in the `.pbix` file.

## Data cleaning (Power Query)

Raw file: 3,097 rows, 18 columns. After cleaning: about 2,960 valid orders.

| Problem | Fix |
|---|---|
| Duplicate orders, IDs with extra spaces or lowercase | Trim, uppercase, remove duplicates on `Order_ID` |
| 4 different date formats, future dates | Custom parsing with fallbacks, filter to 2024-01-01 to 2025-12-31 |
| Test records | Removed rows with `TEST` order IDs |
| Inconsistent text (cities, gender, status, payment, channel) | Mapped all variants to standard values |
| Impossible numbers (negative quantity, age 0 or 199, discount over 100%, price 0, rating 11) | Set to `null`, then fixed or removed |
| Prices inflated 10x or 100x | Detected against the per-product median |
| Wrong totals | Recalculated: price x quantity x (1 - discount) |
| Missing customer attributes | Filled from other orders of the same `Customer_ID`; remaining text gaps labelled `Unknown` |
| Missing rating and shipping days | Left blank on purpose (imputing would distort averages) |

Locale detail: numbers are parsed with the `en-US` culture so decimal points are not misread on a Turkish-locale machine.

## Data model

- `fact_sales`: one row per order
- `Calendar`: date table (2024-2025), related to `fact_sales[Order_date]` (one-to-many), marked as date table

## Main DAX measures

```DAX
Total Orders = DISTINCTCOUNT(fact_sales[Order_ID])

Net Revenue =
CALCULATE(
    SUM(fact_sales[Total_Amount_AZN]),
    fact_sales[Order_Status] <> "Cancelled",
    fact_sales[Order_Status] <> "Returned"
)

Avg Order Value =
DIVIDE(
    [Net Revenue],
    CALCULATE([Total Orders],
        fact_sales[Order_Status] <> "Cancelled",
        fact_sales[Order_Status] <> "Returned")
)

Return Rate =
DIVIDE(CALCULATE([Total Orders], fact_sales[Order_Status] = "Returned"), [Total Orders])

Cancellation Rate =
DIVIDE(CALCULATE([Total Orders], fact_sales[Order_Status] = "Cancelled"), [Total Orders])

Net Revenue LY =
CALCULATE([Net Revenue], SAMEPERIODLASTYEAR('Calendar'[Date]))

Revenue YoY % =
DIVIDE([Net Revenue] - [Net Revenue LY], [Net Revenue LY])

Avg Rating = AVERAGE(fact_sales[Rating])
Avg Shipping Days = AVERAGE(fact_sales[Shipping_Days])
Unique Customers = DISTINCTCOUNT(fact_sales[Customer_ID])
```

## Repository structure

```
├── README.md
├── ecommerce_dashboard.pbix      # Power BI report
├── data/
│   └── ecommerce_sales_raw.csv   # raw (dirty) dataset
├── theme/
│   └── ecommerce_theme.json      # custom Power BI theme
└── screenshots/
    ├── 01_executive_overview.png
    ├── 02_product_customer.png
    └── 03_operations_quality.png
```

## How to use

1. Download `ecommerce_dashboard.pbix` and open it in Power BI Desktop.
2. If the data refresh fails, go to `Transform data` and update the `Source` step to the path of `data/ecommerce_sales_raw.csv` on your machine.
3. Use the Year slicer to compare 2024 and 2025. Year-over-year metrics only show for 2025, because the dataset starts in 2024.

## Author

**Nəsrəddin Vəlizadə**: aspiring data analyst
GitHub: [Valiyevvn](https://github.com/Valiyevvn) · LinkedIn: [nasraddin-valizada](https://www.linkedin.com/in/nasraddin-valizada-36485127b/)
