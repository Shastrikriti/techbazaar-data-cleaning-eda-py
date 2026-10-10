# 🛒 TechBazaar — E-Commerce Sales Data Cleaning & EDA

Turning four messy CSV exports from an Indian electronics store into a clean, analysis-ready dataset and a set of business insights.

> **Author:** Kriti Shastri
> **Dataset name:** *TechBazaar Sales Dataset* (customers · orders · order items · products)
> **Period covered:** 1 Jan 2024 – 15 May 2025 · **Orders:** 1,200 · **Customers:** 500 · **Products:** 80

---

## What this project does

1. **Cleans** four raw tables (duplicates, missing values, inconsistent text, mixed date formats).
2. **Merges** them into a single analysis table (`master_clean.csv`) with no row fan-out.
3. **Analyses** revenue by category, city, time, payment method, product and customer.
4. **Reports** findings and — importantly — the data-quality limits that affect how far you can trust them.

📄 Full write-up: [`docs/REPORT.md`](docs/REPORT.md)

## Headline results

| Metric | Value |
|---|---|
| Total revenue (corrected pipeline) | ₹163.8M (assumed INR) |
| …of which from observed prices | ₹82.9M (50.6%) |
| …of which from imputed prices | ₹80.9M (49.4%) ⚠️ |
| Top category by revenue | Smartphone (31.0%) |
| Top city by revenue | Delhi (28.0%) |
| Most-used payment methods (orders) | UPI 313 · Credit Card 309 · Cash on Delivery 303 |

> ⚠️ Roughly half of all order lines had no recorded unit price, so absolute revenue is an estimate. Rankings are more reliable than absolute values. See [Limitations](docs/REPORT.md#7-limitations--caveats).

## Repository structure

```
techbazaar-sales-insights/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── data/
│   ├── raw/                      # original messy CSVs (untouched)
│   └── cleaned/                  # generated: *_clean.csv + master_clean.csv
├── notebooks/
│   └── cleaning_dataset_original.ipynb   # original exploratory notebook
├── src/
│   └── analysis.py               # reproducible cleaning + analysis pipeline
├── outputs/
│   ├── metrics.json              # headline metrics + cleaning log
│   ├── tables/                   # generated summary tables (CSV)
│   └── charts/                   # generated PNG charts
└── docs/
    └── REPORT.md                 # full structured report
```

## Quick start

```bash
git clone https://github.com/Kriti-Shastri/techbazaar-sales-insights.git
cd techbazaar-sales-insights
python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python src/analysis.py
```

Outputs are written to `data/cleaned/` and `outputs/`. Custom paths:

```bash
python src/analysis.py --raw data/raw --cleaned data/cleaned --out outputs
```

## Data dictionary (raw)

| Table | Rows (raw → clean) | Columns |
|---|---|---|
| `messy_customers.csv` | 520 → 500 | `customer_id`, `customer_name`, `city`, `signup_date` |
| `messy_orders.csv` | 1,240 → 1,200 | `order_id`, `customer_id`, `order_date`, `payment_method` |
| `messy_order_items.csv` | 2,050 → 2,000 | `order_item_id`, `order_id`, `product_id`, `quantity`, `unit_price` |
| `messy_products.csv` | 90 → 80 | `product_id`, `product_name`, `category`, `base_price` |

Relationships: `customers 1─∞ orders 1─∞ order_items ∞─1 products`.

## Cleaning summary

| Problem found | Fix applied |
|---|---|
| 120 fully duplicated rows across tables | Dropped (exact duplicates) |
| City spelled `delhi / DELHI / Delhi`, `Bengaluru` vs `Bangalore` | Title-cased, `Bengaluru → Bangalore` |
| Payment method `upi / UPI`, `credit card / Credit Card`, `Cash / Cash on Delivery` | Standardised to 5 labels |
| Category `smart phone / Smartphone`, `Accessory / Accessories`, case variants | Standardised to 4 categories |
| 3 mixed date formats (`YYYY-MM-DD`, `DD/MM/YYYY`, `MM-DD-YYYY`) | Parsed with explicit rules (not guessed) |
| Missing names / city / payment method | Filled with `Unknown` |
| Missing quantity (318) | Filled with 1, flagged in `quantity_imputed` |
| Missing unit price (1,011 raw) | Product median → overall median, flagged in `price_imputed` |
| Missing base price (57 of 90) | Median (not used in revenue) |

## Charts

| | |
|---|---|
| ![Category](outputs/charts/01_revenue_by_category.png) | ![City](outputs/charts/03_revenue_by_city.png) |
| ![Monthly](outputs/charts/04_monthly_revenue_timeline.png) | ![Quarterly](outputs/charts/05_quarterly_revenue.png) |

## Key insights

- 📱 **Smartphones** are the top category (31% of revenue); **Laptops** the smallest (18%).
- 🏙️ **Delhi** drives 28% of revenue, mostly because it has the most customers (26%).
- 💳 Orders split almost evenly between **UPI, Credit Card and Cash on Delivery**.
- 📉 No clear growth trend across the 17-month window; Q3 2024 was the softest full quarter.
- 🔎 **Data quality is the main story:** 49.5% of order lines had no unit price and 20% of orders have no line items.


## Tech stack

Python 3.10+ · pandas · matplotlib

**Author:** Kriti Shastri 
