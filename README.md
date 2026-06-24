# Part 1 – Retail Orders Data Cleaning and Reporting

## Problem Summary

This project is Part 1 of the capstone assignment and focuses on cleaning, validating, and preparing a retail order-level sales dataset for business analysis. The raw dataset contained common data quality issues such as inconsistent text formatting, mixed date formats, duplicate records, missing values, invalid discounts, and calculation mismatches.

The objective was to produce a clean, analysis-ready dataset along with a data quality report, pivot summary report, screenshots, and documentation of the full cleaning process.

---

# Repository Structure

```bash
part1_data_cleaning/
├── data/
│   ├── raw_orders.xlsx
│   └── cleaned_orders.xlsx
├── outputs/
│   ├── data_quality_report.xlsx
│   ├── pivot_summary.xlsx
│   └── cleaning_log.md
├── screenshots/
│   ├── raw_data_preview.png
│   ├── cleaned_data_preview.png
│   ├── pivot_summary_1.png
│   └── pivot_summary_2.png
└── README.md
```

---

# Dataset Description

The dataset represents retail order-level sales data exported from internal systems. Each row corresponds to an order record and includes customer, location, category, shipping, discount, sales, cost, profit, payment, and order status information.

### Key fields in the dataset:

* `order_id`
* `order_date`
* `ship_date`
* `customer_id`
* `customer_name`
* `segment`
* `region`
* `state`
* `city`
* `category`
* `sub_category`
* `product_name`
* `ship_mode`
* `quantity`
* `unit_price`
* `discount`
* `sales`
* `cost`
* `profit`
* `payment_status`
* `order_status`

The raw file also included a business rules sheet used as reference while cleaning and validating the dataset.

---

# Tools Used

The task required Excel-based final outputs, but Python was used as a supporting tool to perform cleaning, validation, calculations, and report generation efficiently.

### Tools / Libraries used:

* **Google Colab**
* **Python**
* **pandas**
* **numpy**
* **openpyxl**
* **dateutil**

---

# Cleaning Steps Performed

## 1. Preserved Raw Data

* The original file was kept unchanged as `data/raw_orders.xlsx`
* All cleaning was performed on a separate working copy

## 2. Cleaned Text Fields

The following text fields were standardized:

* `customer_name`
* `segment`
* `region`
* `state`
* `city`
* `category`
* `sub_category`
* `ship_mode`
* `payment_status`
* `order_status`

### Actions performed:

* removed leading/trailing spaces
* collapsed multiple spaces
* standardized casing
* replaced separators like underscores / hyphens where required
* normalized category / ship mode / segment text values

## 3. Cleaned and Validated Dates

Both `order_date` and `ship_date` were parsed from mixed date formats into consistent date values.

### Date checks performed:

* missing order date
* missing ship date
* invalid/unrecognized date values
* ship date earlier than order date

### Calculated date field:

* `shipping_delay_days`

## 4. Handled Duplicates

Two types of duplicate checks were performed:

### Exact duplicate rows

* Identified full-row duplicates
* Removed exact duplicate rows while keeping the first occurrence

### Duplicate `order_id` values with conflicting information

* Checked duplicate `order_id` groups for conflicting row values
* Retained such rows instead of deleting them silently
* Flagged them using `duplicate_order_conflict_flag`

## 5. Applied Business Rules

The business rules sheet provided with the assignment was used to implement required handling for missing values, discounts, and order/payment statuses.

### Rules applied:

* Missing `region` → filled as **Unknown**
* Missing `ship_mode` → filled as **Unknown**
* Missing `discount` → treated as **0** for cleaned calculations, but separately flagged
* Negative discounts → flagged as invalid
* Cancelled orders → excluded from final completed-sales summary
* Failed payments → excluded from final completed-sales summary
* Refunded orders → summarized separately
* Ship date before order date → flagged as invalid shipping record

## 6. Created Calculated Columns

The following columns were added to the cleaned dataset:

* `cleaned_discount`
* `calculated_sales`
* `calculated_profit`
* `profit_margin`
* `shipping_delay_days`
* `order_month`
* `order_year`
* `data_quality_flag`

## 7. Validated Sales and Profit

Recalculated sales and profit values were compared against the original dataset values.

### Validation flags created:

* `sales_mismatch_flag`
* `profit_mismatch_flag`

## 8. Built Data Quality Flag

Each row was finally classified into:

* **Clean**
* **Warning**
* **Invalid**

based on the combination of data quality issues present in that record.

---

# Business Rules Applied

## Missing Region

Missing `region` values were filled as **Unknown**.

## Missing Ship Mode

Missing `ship_mode` values were filled as **Unknown**.

## Missing Discount

Missing discount values were treated as **0** for cleaned calculations if required, while also being flagged separately.

## Invalid Discount

Negative discounts were flagged as invalid. A high-discount check was also performed.

## Cancelled / Failed / Refunded Orders

These orders were retained in the cleaned dataset for transparency, but excluded from the main completed-sales reporting dataset:

* Cancelled orders
* Failed payments
* Refunded orders

## Ship Date Before Order Date

Rows where `ship_date < order_date` were flagged as invalid shipping records.

---

# Summary of Data Quality Issues Found

## Duplicate Issues

* Exact duplicate rows found: **20**
* Exact duplicate rows removed: **20**
* Duplicate `order_id` groups found: **31**
* Conflicting duplicate `order_id` groups found: **12**
* Rows flagged as conflicting duplicate records: **24**

## Missing / Discount Issues

* Missing raw discount values: **18**
* Negative discount rows: **15**
* High discount rows: **0**

## Date Issues

* Missing order dates: **0**
* Missing ship dates: **0**
* Invalid order dates: **0**
* Invalid ship dates: **0**
* Ship date before order date: **93**

## Order / Payment Status Issues

* Cancelled orders: **145**
* Failed payments: **69**
* Refunded orders: **71**

## Calculation Mismatch Issues

* Sales mismatch rows: **61**
* Profit mismatch rows: **61**

## Final Record Quality Summary

* **Clean:** 567 rows
* **Warning:** 239 rows
* **Invalid:** 106 rows

---

# Summary of Final Pivot Reports

The file `outputs/pivot_summary.xlsx` contains the following pivot summaries:

## 1. Sales and Profit by Region

Summarizes total calculated sales and total calculated profit across regions.

## 2. Sales and Profit by Category and Sub-Category

Shows the most profitable and highest-selling category/sub-category combinations.

## 3. Order Count by Ship Mode

Provides order distribution across shipping methods.

## 4. Profit Margin by Customer Segment

Compares average profit margin across customer segments.

## 5. Refunded / Cancelled / Failed Orders by Region

Summarizes issue orders across regions for operational review.

## 6. Monthly Sales Trend

Shows calculated sales month-wise across available years in the dataset.

At least two pivot outputs were sorted for better readability and evaluation.

---

# Key Business Insights

Based on the cleaned and summarized data:

1. **South, West, and East regions contribute strongly to sales and profit**, while `Unknown` region records exist due to missing original region values.
2. **Technology and Furniture sub-categories such as Copiers, Chairs, Phones, and Furnishings show strong profitability**, indicating high-value product lines.
3. **Standard Class and First Class shipping contribute a large share of total order volume**, making shipping mode an important operational dimension.
4. **A noticeable number of cancelled, refunded, and failed-payment orders exist**, which may impact final business reporting if not filtered correctly.
5. **Sales/profit mismatches and conflicting duplicate order records suggest upstream system inconsistencies** that should be reviewed by business operations or source-system owners.
6. **93 rows with ship dates before order dates** indicate clear data quality or process-entry issues that require validation at source.

---

# Assumptions and Limitations

## Assumptions

1. Missing `region` and `ship_mode` values were filled with **Unknown** as per the provided business rules.
2. Missing discounts were treated as **0** for cleaned calculations but tracked separately using a flag.
3. Conflicting duplicate `order_id` records were retained and flagged instead of being force-deleted.
4. Cancelled, failed, and refunded orders were excluded from the main reporting subset but kept in the cleaned file for traceability.

## Limitations

1. The cleaning process flags problematic records but does not attempt to manually determine the “true” business value for conflicting duplicate orders.
2. Recalculated sales and profit were created for validation, but the original raw columns were not overwritten.
3. Some quality rules depend on the assumptions defined in the provided business rule sheet rather than external domain validation.

---

# Screenshots Included

The `screenshots/` folder contains the following proof screenshots:

* `raw_data_preview.png` → preview of raw dataset before cleaning
* `cleaned_data_preview.png` → cleaned dataset with calculated columns and quality flags
* `pivot_summary_1.png` → pivot outputs for region and category/sub-category summaries
* `pivot_summary_2.png` → pivot outputs for issue orders by region and monthly sales trend

---

# Final Deliverables Included

This repository includes all required outputs for Part 1:

* Raw dataset → `data/raw_orders.xlsx`
* Cleaned dataset → `data/cleaned_orders.xlsx`
* Data quality report → `outputs/data_quality_report.xlsx`
* Pivot summary report → `outputs/pivot_summary.xlsx`
* Cleaning log → `outputs/cleaning_log.md`
* Screenshots → `screenshots/`
* README → `README.md`

---

# Conclusion

This project cleaned and validated the provided retail order dataset, applied the required business rules, created calculated columns, documented quality issues, and produced reporting-ready Excel outputs. The final cleaned dataset is structured for business review and future analytical use while preserving visibility into data quality concerns through flags and supporting reports.


