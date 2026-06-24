# Cleaning Log – Part 1 Data Cleaning

## 1. Objective

The goal of this task was to clean, validate, and prepare the provided retail orders dataset for analysis while preserving the original raw file. The final deliverables include a cleaned Excel dataset, a data quality report, pivot summary report, screenshots, and this cleaning log.

---

## 2. Files Used

* **Raw dataset:** `data/raw_orders.xlsx`
* **Cleaned dataset:** `data/cleaned_orders.xlsx`
* **Data quality report:** `outputs/data_quality_report.xlsx`
* **Pivot summary report:** `outputs/pivot_summary.xlsx`

---

## 3. Tools Used

* **Google Colab / Python**
* **pandas**
* **numpy**
* **openpyxl**
* **dateutil**

Python was used as a supporting tool to clean, validate, calculate, and export the required Excel-based outputs as permitted by the faculty, while ensuring that the final submission remains in the required Excel file format.

---

## 4. Cleaning Steps Performed

### 4.1 Preserved Raw Data

* Kept the original file unchanged as `data/raw_orders.xlsx`.
* Created a working copy in Python for all cleaning and transformation steps.

---

### 4.2 Text Field Cleaning

The following text fields were cleaned and standardized:

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

#### Cleaning actions applied:

* Removed leading and trailing spaces
* Collapsed multiple spaces into a single space
* Replaced underscores / hyphens where applicable
* Standardized text casing using Title Case
* Corrected category / segment / ship mode naming inconsistencies where possible

---

### 4.3 Date Cleaning and Validation

The following date fields were cleaned:

* `order_date`
* `ship_date`

#### Date parsing approach:

A custom date parser was used to handle multiple formats such as:

* `21 Jul 2024`
* `07/27/2024`
* `05 Sep 2024`
* `28-11-2024`
* `2025-09-01`

#### Date validations performed:

* Checked for missing order dates
* Checked for missing ship dates
* Checked for invalid/unrecognized date values
* Flagged rows where `ship_date < order_date`

#### Calculated date column created:

* `shipping_delay_days = ship_date - order_date`

---

## 5. Duplicate Handling

### 5.1 Exact Duplicate Rows

* Exact duplicate rows were identified using full-row comparison.
* These rows were removed from the cleaned dataset while keeping the first occurrence.

### 5.2 Duplicate `order_id` Values with Conflicting Information

* Duplicate `order_id` groups were checked for conflicting row values.
* If rows shared the same `order_id` but had different values in other columns, they were **not deleted silently**.
* Such rows were flagged using:

  * `duplicate_order_conflict_flag = True`

This ensures that potentially important conflicting business records remain visible for evaluator review.

---

## 6. Business Rules Applied

The following business rules were implemented:

### 6.1 Missing Region

* Missing `region` values were filled as **"Unknown"**
* Such cases were counted in the quality reporting process

### 6.2 Missing Ship Mode

* Missing `ship_mode` values were filled as **"Unknown"**
* Such cases were counted in the quality reporting process

### 6.3 Missing Discount

* Missing raw discount values were treated as **0 only for cleaned/calculated usage**
* The original missing condition was separately tracked using:

  * `missing_discount_flag`

### 6.4 Invalid Discount Rules

* Negative discounts were flagged using:

  * `negative_discount_flag`
* Very high discounts above the accepted threshold were checked using:

  * `high_discount_flag`

### 6.5 Order / Payment Status Rules

The following flags were created:

* `cancelled_order_flag`
* `failed_payment_flag`
* `refunded_order_flag`

#### Business interpretation used:

* **Cancelled orders** should not contribute to final completed sales summary
* **Failed payments** should not contribute to final completed sales summary
* **Refunded orders** must be summarized separately
* These records were excluded from the main reporting dataset used for clean sales pivots

### 6.6 Invalid Shipping Records

* Rows where `ship_date < order_date` were flagged as invalid shipping records

---

## 7. Calculated Columns Created

The following calculated columns were created in the cleaned dataset:

* `cleaned_discount` → standardized discount value used in calculations
* `calculated_sales` → `quantity × unit_price × (1 - cleaned_discount)`
* `calculated_profit` → `calculated_sales - cost`
* `profit_margin` → `calculated_profit / calculated_sales`
* `shipping_delay_days` → days between order and ship date
* `order_month` → extracted from `order_date`
* `order_year` → extracted from `order_date`
* `data_quality_flag` → final record quality classification

---

## 8. Validation Checks Performed

### 8.1 Sales and Profit Recalculation

The original `sales` and `profit` columns were compared against recalculated values.

#### Flags created:

* `sales_mismatch_flag`
* `profit_mismatch_flag`

These were used to identify rows where stored values did not match the values derived from business logic.

---

## 9. Final Data Quality Flag Logic

A final `data_quality_flag` column was assigned to each row using the following logic:

### Invalid

Rows marked **Invalid** if any of the following were true:

* Negative discount
* High/invalid discount
* Ship date before order date

### Warning

Rows marked **Warning** if any of the following were true:

* Duplicate order conflict
* Missing original discount
* Sales mismatch
* Profit mismatch
* Cancelled order
* Failed payment
* Refunded order

### Clean

Rows marked **Clean** if none of the above issues were present.

---

## 10. Summary of Issues Found

### Missing / Data Quality Issues

* Missing raw discount values found: **18**
* Missing region values were filled as **Unknown**
* Missing ship mode values were filled as **Unknown**

### Duplicate Issues

* Exact duplicate rows found: **20**
* Exact duplicate rows removed: **20**
* Duplicate `order_id` groups found: **31**
* Conflicting duplicate `order_id` groups found: **12**
* Rows flagged as conflicting duplicate records: **24**

### Date Issues

* Missing order dates: **0**
* Missing ship dates: **0**
* Invalid order dates: **0**
* Invalid ship dates: **0**
* Ship date before order date: **93**

### Discount / Calculation Issues

* Negative discount rows: **15**
* High discount rows: **0**
* Sales mismatch rows: **61**
* Profit mismatch rows: **61**

### Order / Payment Status Issues

* Cancelled orders: **145**
* Failed payments: **69**
* Refunded orders: **71**

### Final Quality Classification

* **Clean:** 567 rows
* **Warning:** 239 rows
* **Invalid:** 106 rows

---

## 11. Assumptions Made

1. Missing `region` values were filled with **Unknown** as per the business rule sheet.
2. Missing `ship_mode` values were filled with **Unknown** as per the business rule sheet.
3. Missing discount values were treated as **0** for calculation purposes, but also flagged separately for reporting.
4. Conflicting duplicate `order_id` rows were retained and flagged rather than deleted, because they may represent business-important inconsistencies.
5. Cancelled, failed, and refunded orders were excluded from the main completed-sales pivot reporting dataset, but retained in the cleaned file for traceability and issue reporting.
6. A practical high-discount threshold check was applied, although no such rows were found in this dataset.

---

## 12. Records Removed / Flagged

### Removed

* **20 exact duplicate rows removed**

### Flagged

* **24 rows flagged as conflicting duplicate order records**
* **93 rows flagged where ship date was before order date**
* **15 rows flagged for negative discount**
* **61 rows flagged for sales mismatch**
* **61 rows flagged for profit mismatch**
* Cancelled / failed / refunded orders flagged for separate reporting treatment

---

## 13. Limitations

1. The cleaning process standardizes and flags issues based on the rules provided, but it does not infer the “correct” business value where conflicting duplicate records exist.
2. Rows flagged for sales/profit mismatch were not force-corrected in the original raw fields; instead, recalculated fields were created and mismatches were documented.
3. “Unknown” values were used only where explicitly allowed by the provided business rules.
4. The final outputs are evaluator-friendly and Excel-based, but the cleaning logic itself was executed in Python for speed, consistency, and reproducibility.

---

## 14. Final Output Generated

The following outputs were created successfully:

* `data/cleaned_orders.xlsx`
* `outputs/data_quality_report.xlsx`
* `outputs/pivot_summary.xlsx`
* `screenshots/raw_data_preview.png`
* `screenshots/cleaned_data_preview.png`
* `screenshots/pivot_summary_1.png`
* `screenshots/pivot_summary_2.png`

This completes the Part 1 data cleaning workflow.


