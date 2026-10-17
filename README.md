# TechBazaar: E-commerce Sales Data Cleaning & EDA

Cleaning and exploratory analysis of messy e-commerce sales data (customers, orders, order items and products) using Python, pandas, matplotlib and seaborn.

📄 **Documentation:** [`TechBazaar_Documentation.pdf`](TechBazaar_Documentation.pdf) · [`TechBazaar_Documentation.docx`](TechBazaar_Documentation.docx)
📓 **Notebook:** [`Cleaning_TechBazaar.ipynb`](Cleaning_TechBazaar.ipynb)

## Overview

TechBazaar sells smartphones, tablets, laptops and accessories to customers across Indian cities. The raw exports contain inconsistent spellings, three different date formats, missing values and duplicate rows. This project cleans the four tables, merges them into one dataset, and answers key business questions about revenue, products, payments and cities.

## Objectives

- Clean and standardise four raw CSV files
- Parse all order dates correctly across three mixed formats
- Remove customers with no city and orders with no payment method, keeping all tables consistent
- Engineer features: Month, Quarter, Year and line-item `Amount` (quantity × unit price)
- Analyse revenue by category, payment method, year, quarter, month and city

## Dataset

| File | Raw rows | Final rows | Columns |
|------|---------:|-----------:|---------|
| `messy_customers.csv` | 520 | 460 | customer_id, customer_name, city, signup_date |
| `messy_orders.csv` | 1,240 | 958 | order_id, customer_id, order_date, payment_method |
| `messy_order_items.csv` | 2,050 | 1,624 | order_item_id, order_id, product_id, quantity, unit_price |
| `messy_products.csv` | 90 | 80 | product_id, product_name, category, base_price |

Relationships: `customers.customer_id → orders.customer_id`, `orders.order_id → order_items.order_id`, `order_items.product_id → products.product_id`. The merged dataset has **1,624 rows × 22 columns**.

## Data cleaning steps

- Removed **41** customer rows with no city and **145** order rows with no payment method; missing customer names set to `Unknown`
- Standardised city names (Delhi/delhi/DELHI → Delhi, Bengaluru → Bangalore), payment methods (4 labels) and product categories (4 labels)
- Removed duplicates in every table
- Parsed order dates explicitly for each of the three formats (`%Y-%m-%d`, `%d/%m/%Y`, `%m-%d-%Y`), with no unparsed values
- Dropped orders of removed customers and order items of removed orders, so every result uses the same rows
- Filled missing quantity with 1 (zeros set to 1), missing unit price with the product median, and missing base price with the mean
- Extracted Month, Quarter and Year and merged all tables into one dataset

## Key insights

All figures use the same 1,624 order items and add up to **₹145.14M** (≈ ₹14.51 crore).

- **Top category:** Smartphone, with 1,033 units and ₹43.60M revenue (30.0%), followed by Accessories and Tablet
- **Payment methods:** Cash on Delivery (283 orders), Credit Card (277) and UPI (274) are nearly tied; Debit Card is far behind (124)
- **Top city:** Delhi with ₹44.41M (30.6% of revenue), followed by Mumbai (₹26.19M) and Hyderabad (₹21.41M)
- **Quarters:** Q1 is highest (₹53.71M) and Q3 lowest (₹23.25M)
- **Months:** April and January are highest on average, November is lowest
- **Years:** 2024 ₹104.83M, 2025 ₹40.31M (2025 covers only 1 Jan – 15 May)

## Assumptions & limitations

- Rows with no city or no payment method were removed, so results cover 958 of 1,200 unique orders and 1,624 of 2,000 order items.
- "Cash" and "Cash on Delivery" were treated as the same payment method.
- About half of the unit prices (1,011 of 2,050) were filled with a median, which flattens price differences.
- Orders span 1 Jan 2024 – 15 May 2025, so January–May appear twice and June–December once; monthly figures are averaged per calendar month.
- Product names and categories are inconsistent in the source (e.g. "Wireless Mouse" listed as Laptop).

## Project structure

```
├── Cleaning_TechBazaar.ipynb          # cleaning + EDA notebook
├── TechBazaar_Documentation.pdf       # full written documentation
├── TechBazaar_Documentation.docx      # editable Word version
├── messy_customers.csv
├── messy_orders.csv
├── messy_order_items.csv
├── messy_products.csv
└── README.md
```

## How to run

```bash
pip install pandas matplotlib seaborn jupyter
jupyter notebook Cleaning_TechBazaar.ipynb
```

Keep the four CSV files in the same folder as the notebook, and run all cells from top to bottom.

## Tools

Python · pandas · matplotlib · seaborn · Jupyter Notebook
