# Olist Delivery KPIs & Seller Defect Rate (Python)

An end-to-end analysis of **96,447 delivered orders** from a Brazilian online marketplace: data quality checks, table joins with row-count control, delivery KPIs, the link between late delivery and customer reviews, and a **DPMO-style defect rate per seller** with an audit list. The pipeline ends in an **automatic Excel report** with charts.

**Tools:** Python · pandas · matplotlib · openpyxl
**Data:** [Brazilian E-Commerce Public Dataset by Olist](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) (Kaggle, 2016–2018)

> **About this project:** my background is SQL (Amazon Redshift), Excel/VBA and Power BI. This is my first Python project. I built it step by step to move my day-to-day analysis methods into Python, including a defect-rate (DPMO) approach I used in operations.

---

## Key findings

| | |
|---|---|
| **93.2%** of orders arrive on time | Late orders are late by **10.6 days** on average |
| On-time orders arrive **13.5 days early** on average | The promised delivery date is very cautious |
| Late orders get an average review of **2.27** vs **4.29** | **62%** of late orders get a 1–2 star review (vs 9% on time) |
| Late orders are **6.7%** of rated orders | …but cause **32.5%** of all bad reviews, about **5x** their share |
| Worst months: **Nov 2017** (Black Friday, 12.4% late) and **Feb–Mar 2018** (14.1% and 19.0%) | From Apr 2018 late rate and delivery time both improve |
| North-east states are worst (**MA 17.4%**, CE 13.8%, BA 12.2%) | **RJ** is an outlier: 12.1% late with only 14.8 days delivery, so distance alone does not explain it |
| **83 of 615** sellers flagged for audit | They cause **20%** of defects: seller audits help, but most defects need system-level fixes |

![Review score by delay](output/charts/03_review_by_delay.png)

---

## How it works

| Step | Script | What it does |
|---|---|---|
| 1 | `01_load_and_check.py` | First look: size of each table, missing values, duplicate keys |
| 2 | `02_quality_checks.py` | Checks that should return zero; every finding saved to a quality log |
| 3 | `03_join_tables.py` | Chooses the orders to analyse, joins tables with `validate=` and a row count after every join, adds delay measures |
| 4 | `04_delivery_kpis.py` | KPIs overall, by state, by month and by seller (minimum group sizes) |
| 5 | `05_review_impact.py` | Review score and bad-review rate by delay range |
| 6 | `06_seller_dpmo.py` | Seller defect rate (DPMO), audit threshold, ranking by excess defects |
| 7 | `07_charts.py` | 4 charts (PNG) |
| 8 | `08_excel_report.py` | Excel report: one sheet per topic, formatted, with charts |

`load_data.py` holds the shared code that reads the CSV files and turns date columns into real dates.

## Data quality decisions

| Finding | Rows | Decision |
|---|---|---|
| Orders not delivered (shipped, canceled, …) | ~2,960 | Left out of delivery KPIs: they cannot be on time or late |
| Status "delivered" but no delivery date | 8 | Left out |
| Given to the carrier **after** reaching the customer | 23 | Left out: impossible order of dates |
| Orders with more than one review | 547 | Kept the **latest** review, so the join does not duplicate orders |
| Orders with more than one seller | 1,278 | Kept in delivery KPIs; **left out of the seller score**, because one delivery date cannot be split between sellers |

Row check: 96,478 delivered − 8 − 23 = **96,447** orders in the analysis.

## Definitions

- **Late:** delivered on a later **day** than the estimated delivery date (time of day ignored, because the estimate has no time).
- **Bad review:** 1 or 2 stars.
- **Seller DPMO:** each single-seller order has 2 opportunities for a defect: late delivery, and a bad review (if the customer left a review).
  `DPMO = defects / opportunities × 1,000,000`
- **Network DPMO (95,900):** all defects ÷ all opportunities, **not** the average of seller DPMOs.
- **Audit flag:** seller DPMO above **1.5 × network DPMO (143,850)** and at least **30 orders**.
- **Excess defects:** seller defects minus the defects expected at the network rate for that seller's volume. The audit list is sorted by this, so the audit goes where it removes the most problems.

![Seller DPMO](output/charts/04_seller_dpmo.png)

## Limits

- The link between delay and reviews is strong and step by step, but it is a link, not proof of cause: a bad review can also be about the product.
- Small sellers vary more by chance (see the funnel shape above). A flag should be confirmed in a second period before an audit.
- The data does not explain the Feb–Mar 2018 peak: that is a question for the logistics team, not a conclusion.

## How to run

```
python -m venv .venv
.venv\Scripts\activate          (Mac/Linux: source .venv/bin/activate)
pip install -r requirements.txt
```
Download the dataset from Kaggle and put the CSV files in `data/raw/`, then:
```
python run_all.py
```
Result: `output/olist_delivery_report.xlsx` and the charts in `output/charts/`.
