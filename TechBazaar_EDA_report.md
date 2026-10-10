# TechBazaar Sales Dataset — Cleaning & Exploratory Analysis Report

**Author:** Kriti Shastri
**Scope:** 1 Jan 2024 – 15 May 2025 · **Source:** four messy CSV exports · **Currency:** not stated in the data; INR assumed (Indian cities, UPI payments)
**Code:** [`src/analysis.py`](../src/analysis.py) · **Numbers:** [`outputs/metrics.json`](../outputs/metrics.json), [`outputs/tables/`](../outputs/tables)

---

## Contents

1. [Executive summary](#1-executive-summary)
2. [Dataset overview](#2-dataset-overview)
3. [Data quality audit & cleaning](#3-data-quality-audit--cleaning)
4. [Methodology](#4-methodology)
5. [Findings](#5-findings)
6. [Recommendations](#6-recommendations)
7. [Limitations & caveats](#7-limitations--caveats)
8. [Changes vs. the original notebook](#8-changes-vs-the-original-notebook)
9. [Reproducing the analysis](#9-reproducing-the-analysis)

---

## 1. Executive summary

- The business generated an estimated **₹163.8M** from 1,200 orders and 500 registered customers (448 of whom ordered).
- **Smartphones** lead on revenue (31%) and units; **Laptops** are last (18%).
- **Delhi** contributes 28% of revenue, more than double the next city (Mumbai, 16%).
- Payments are split almost evenly between **UPI, Credit Card and Cash on Delivery** (~25% of orders each); Debit Card is the smallest named method.
- Revenue was ~₹33.8M in Q1 2024 and ~₹28–31M in most later full quarters, dipping to ₹26.1M in Q3 2024. There is **no clear growth trend**.
- **Trust caveat:** 49.5% of order lines had no unit price, so about half of the revenue figure is imputed. 20% of orders have no line items at all. Use rankings and shares with moderate confidence; do not quote the absolute total as audited revenue.

## 2. Dataset overview

| Table | Raw rows | Clean rows | Key |
|---|---|---|---|
| customers | 520 | 500 | `customer_id` |
| orders | 1,240 | 1,200 | `order_id` |
| order_items | 2,050 | 2,000 | `order_item_id` |
| products | 90 | 80 | `product_id` |

```
customers ─1:N─ orders ─1:N─ order_items ─N:1─ products
```

Referential integrity is good: every order item points to an existing order and product, and every order to an existing customer. The merge is validated as many-to-one at each step and the row count is asserted unchanged (2,000).

## 3. Data quality audit & cleaning

### 3.1 Issues found

| Table | Issue | Count |
|---|---|---|
| customers | Exact duplicate rows | 20 |
| customers | Missing name / missing city | 28 / 41 |
| customers | City case & spelling variants | 12 labels → 7 + Unknown |
| orders | Exact duplicate rows | 40 |
| orders | Missing payment method | 145 |
| orders | Payment label variants | 8 labels → 4 + Unknown |
| orders | Mixed date formats | 3 formats |
| order_items | Exact duplicate rows | 50 |
| order_items | Missing quantity | 318 |
| order_items | Missing unit price | 1,011 raw (49.5% after de-dup) |
| order_items | Zero quantity | 341 raw (328 after de-dup) |
| products | Exact duplicate rows | 10 |
| products | Missing base price | 57 of 90 |
| products | Category label variants | 8 labels → 4 |

### 3.2 Treatment

| Decision | Rationale |
|---|---|
| Drop exact duplicate rows in **all four** tables | They are verbatim repeats of earlier rows (same ID, same values). |
| Parse dates by explicit rule: `YYYY-MM-DD` ISO; `DD/MM/YYYY` for slashes; `MM-DD-YYYY` for dashes | Rows such as `13/04/2025` (day > 12) and `04-26-2025` (month-first) prove each convention. Letting pandas guess per row would misread ambiguous values like `09/05/2024`. |
| Treat `Cash` and `Cash on Delivery` as one method | Assumed to mean the same thing; flagged as an assumption. |
| Missing quantity → 1 | An order line implies at least one unit. Flagged in `quantity_imputed`. |
| Missing unit price → median of that product's observed prices, else overall median; flagged in `price_imputed` | Median is robust to outliers; the flag lets the analysis separate observed from estimated revenue. |
| Keep zero-quantity lines (328) | They contribute ₹0 and 0 units; kept visible rather than silently rewritten. |
| Missing text fields → `Unknown` | Keeps rows in the analysis; `Unknown` is reported as its own bucket. |

## 4. Methodology

- Revenue = `quantity × unit_price` per order line.
- Time grouping uses the **order** date (not signup date).
- Quarterly and monthly views are shown **chronologically** with the year, never pooled across years, except the explicit seasonality view, which uses a per-month *average*.
- Metrics that depend on imputed prices are reported both in total and split by `price_imputed`.

## 5. Findings

### 5.1 Headline numbers

| Metric | Value |
|---|---|
| Total revenue | ₹163,782,246 |
| From observed prices | ₹82,878,352 (50.6%) |
| From imputed prices | ₹80,903,894 (49.4%) |
| Units sold | 3,685 |
| Order lines | 2,000 |
| Average order value (orders that have line items) | ₹170,607 |
| Customers who ordered | 448 of 500 (89.6%) |

### 5.2 Revenue by product category

| Category | Revenue | Share | Units |
|---|---|---|---|
| Smartphone | ₹50.7M | 31.0% | 1,187 |
| Accessories | ₹43.3M | 26.5% | 938 |
| Tablet | ₹40.1M | 24.5% | 852 |
| Laptop | ₹29.6M | 18.1% | 708 |

![Revenue by category](../outputs/charts/01_revenue_by_category.png)

Using **observed prices only**, the order is Smartphone ₹26.3M > Accessories ₹22.5M > Tablet ₹20.1M > Laptop ₹13.9M, so the ranking holds either way.

### 5.3 Revenue by city

| City | Revenue | Share |
|---|---|---|
| Delhi | ₹45.9M | 28.0% |
| Mumbai | ₹26.4M | 16.1% |
| Hyderabad | ₹22.3M | 13.6% |
| Bangalore | ₹20.1M | 12.3% |
| Pune | ₹14.2M | 8.7% |
| Unknown | ₹13.4M | 8.2% |
| Kolkata | ₹11.2M | 6.9% |
| Chennai | ₹10.3M | 6.3% |

![Revenue by city](../outputs/charts/03_revenue_by_city.png)

Delhi also has the most customers (131 of 500, 26%), so its revenue lead mostly reflects customer count rather than higher spend per customer.

### 5.4 Revenue over time

| Quarter | Revenue |
|---|---|
| 2024 Q1 | ₹33.8M |
| 2024 Q2 | ₹31.4M |
| 2024 Q3 | ₹26.1M |
| 2024 Q4 | ₹28.9M |
| 2025 Q1 | ₹27.9M |
| 2025 Q2 *(to 15 May only)* | ₹15.6M |

Calendar years: 2024 = ₹120.2M; 2025 (Jan – mid-May) = ₹43.6M. **2025 is a partial year** and must not be compared with 2024 as a decline.

![Quarterly revenue](../outputs/charts/05_quarterly_revenue.png)
![Monthly timeline](../outputs/charts/04_monthly_revenue_timeline.png)

Seasonality (average revenue per calendar month; partial May 2025 excluded): strongest in Jan, Mar, Apr and Dec (≈₹11.0–11.4M); weakest in Aug (₹6.9M), Nov (₹7.5M) and Feb (₹8.3M). Each month has only one or two observations, so treat this as indicative only.

![Seasonality](../outputs/charts/07_seasonality_by_month.png)

### 5.5 Payment methods

| Method | Orders | Revenue |
|---|---|---|
| UPI | 313 | ₹41.4M |
| Credit Card | 309 | ₹41.2M |
| Cash on Delivery | 303 | ₹42.2M |
| Unknown | 141 | ₹18.1M |
| Debit Card | 134 | ₹20.9M |

Order counts are for all 1,200 orders; revenue only covers orders that have line items.

![Orders by payment](../outputs/charts/06_orders_by_payment_method.png)

### 5.6 Top products and customers

Top products by revenue (by `product_id`): P076 Wireless Mouse ₹5.7M, P052 Xiaomi Redmi ₹5.1M, P072 Dell Inspiron ₹4.7M, P017 Keyboard ₹4.2M, P033 Wireless Mouse ₹4.1M. Full list: [`top_products.csv`](../outputs/tables/top_products.csv).

Top customer: C0218 with ₹1.66M across 6 orders ([`top_customers.csv`](../outputs/tables/top_customers.csv)).

## 6. Recommendations

1. **Fix price capture at source.** Half of order lines lack a unit price; this is the single biggest limit on trustworthy revenue reporting.
2. **Enforce controlled vocabularies** (dropdowns / enums) for city, payment method and category to stop label drift.
3. **Standardise date formats** (ISO 8601) at ingestion.
4. **Investigate the 240 orders with no line items** (20% of orders) and the 328 zero-quantity lines: failed checkouts, cancellations, or an export gap?
5. **Fix the product master.** Product names and categories conflict (e.g. "Wireless Mouse" listed as Laptop, "iPhone 13" as Tablet) and one product name maps to several IDs, so category-level conclusions depend on category labels that may be wrong.
6. **Business actions to test:** deepen Delhi/Mumbai inventory and campaigns; run targeted promotions in Q3 and Nov–Feb, the softer periods; prioritise Smartphones and Accessories in assortment planning.

## 7. Limitations & caveats

- **Imputed prices (49.5% of lines).** Revenue totals, AOV and per-product/customer figures embed estimates. Rankings were cross-checked with observed prices only for categories; other cuts were not.
- **Unit prices appear unrelated to product.** The same product shows widely different prices across lines, and `base_price` is 63% missing, so price imputation cannot be validated against a reference list.
- **Orders without items (240).** They contribute no revenue and are excluded from AOV; the real order count with revenue is 960.
- **Category/name mismatch** (see Recommendation 5).
- **Partial final period** (2025 ends 15 May).
- **Currency and `Cash` ≈ `Cash on Delivery`** are assumptions.
- The data looks synthetic/anonymised (`Customer_N` names); conclusions are illustrative.

## 8. Changes vs. the original notebook

The original notebook (`notebooks/cleaning_dataset_original.ipynb`) is a good first pass. Reviewing it surfaced issues now fixed in `src/analysis.py`:

| # | Issue in notebook | Impact | Fix |
|---|---|---|---|
| 1 | `orders.drop_duplicates()` and `order_items.drop_duplicates()` were called **without `inplace=True` or assignment** | 40 duplicate orders and 50 duplicate items stayed in the data; merged table had 2,190 rows instead of 2,000 | De-duplicate and assign for all tables |
| 2 | Revenue totals used the duplicated data (₹169.5M from items; ₹181.4M from the merged table, which don't agree with each other) | Revenue overstated and internally inconsistent | One validated merge; corrected total ₹163.8M |
| 3 | Payment labels `Cash On Delivery` vs `Cash on Delivery` remained separate after `.str.title()` | Cash on Delivery split into 148 + 170 instead of 318 and ranked wrongly | Lower-case first, then map to canonical labels |
| 4 | `quantity.replace({"0": "1"})` on a numeric column | No effect (strings never match floats); 341 zero quantities remained silently | Zeros explicitly kept and reported |
| 5 | `fillna(45057)` after printing a mean of 45,047 | Typo in the imputed value (small) | Median computed in code, not hand-typed |
| 6 | Unit price filled with one global mean for ~1,000 lines | Flattens price differences; no way to see how much revenue is estimated | Product-level median + `price_imputed` flag + split reporting |
| 7 | `pd.to_datetime(format="mixed")` | Ambiguous dates such as `09/05/2024` can be misread | Explicit per-format parsing |
| 8 | Quarterly/monthly series sorted by revenue, plotted as a line; monthly totals pooled across years | Misleading ordering; Jan–May counted twice (17-month span) | Chronological order; per-month average for seasonality |
| 9 | Payment chart titled "Total revenue per payment method" but plotted order counts | Wrong label | Charts titled correctly |
| 10 | Merge did not validate cardinality | Risk of silent row fan-out | `validate="m:1"` plus row-count assert |

Findings that **did not change**: Smartphone is the top category and Delhi the top city.

## 9. Reproducing the analysis

```bash
pip install -r requirements.txt
python src/analysis.py
```

Generates `data/cleaned/*.csv`, `outputs/tables/*.csv`, `outputs/charts/*.png` and `outputs/metrics.json`.

---

*Prepared by Kriti Shastri. Every figure above comes from the raw CSVs in `data/raw/` and is reproducible with `src/analysis.py`.*
