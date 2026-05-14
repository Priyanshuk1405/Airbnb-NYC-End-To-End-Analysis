# 🏠 Airbnb NYC — End-to-End Data Analysis Project

![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-336791?logo=postgresql&logoColor=white)
![Tableau](https://img.shields.io/badge/Tableau-Dashboard-E97627?logo=tableau&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458?logo=pandas)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

A complete end-to-end data analysis project on **98,109 Airbnb listings in New York City** — covering Python data cleaning, PostgreSQL SQL analysis, and an interactive Tableau dashboard.

> **Live Dashboard →** [Tableau Public](#) *(replace with your link after publishing)*
> **Dataset →** [Airbnb Open Data on Kaggle](https://www.kaggle.com/datasets/arianazmoudeh/airbnbopendata)

---

## 📊 Dashboard Preview

![Airbnb NYC Booking Analysis Dashboard](docs/dashboard_preview.png)

*Interactive dashboard with 6 charts, KPI cards, NYC map, and cross-chart filters*

---

## 🔑 Key Findings

| Finding | Result |
|---|---|
| Highest demand borough | **Brooklyn** — 242 avg booked days/year (66.4% occupancy) |
| Lowest demand borough | **Staten Island** — 169 avg booked days/year (46.4% occupancy) |
| Most booked room type | **Private room** — 234 days/year |
| Least booked room type | **Hotel room** — 145 days/year |
| Top neighbourhood | **Navy Yard, Brooklyn** — 296 booked days/year |
| Impact of instant booking | Minimal — only +1 day/year difference |
| Impact of price on demand | Very weak — all price ranges book ~228–233 days/year |
| Primary demand driver | **Location** — borough and neighbourhood, not price or policy |

---

## 📁 Project Structure

```
airbnb-nyc-analysis/
│
├── data/
│   ├── Airbnb_Open_Data.csv           ← raw dataset (download from Kaggle)
│   └── Airbnb_Tableau_Ready.csv       ← cleaned dataset for Tableau
│
├── notebooks/
│   ├── Airbnb_EDA_Cleaning.ipynb      ← Python EDA & data cleaning
│   └── Airbnb_SQL_Analysis.ipynb      ← SQL queries + Python visualisations
│
├── sql/
│   └── booking_analysis_queries.sql   ← all SQL queries (standalone)
│
├── dashboard/
│   └── Airbnb_Dashboard.twbx          ← Tableau packaged workbook
│
├── docs/
│   ├── Airbnb_Project_Report.docx     ← full project documentation
│   └── dashboard_preview.png          ← dashboard screenshot
│
├── README.md
├── requirements.txt
└── .gitignore
```

---

## 🛠 Technology Stack

| Layer | Tool | Purpose |
|---|---|---|
| Data Cleaning & EDA | Python, Pandas, NumPy | Load, inspect, clean 102K rows |
| Visualisation | Matplotlib, Seaborn | EDA charts during cleaning |
| Database | PostgreSQL | Store cleaned data, run SQL analysis |
| SQL Analysis | SQL via SQLAlchemy | 8 booking analysis queries |
| Dashboard | Tableau Public | Interactive 6-chart dashboard |
| Documentation | Jupyter Notebook, Word | Project report and notebooks |

---

## 🔄 Project Workflow

```
Raw CSV (102,599 rows)
        ↓
Python EDA & Cleaning (Jupyter)
  - Drop 5 useless columns
  - Remove 541 duplicates
  - Fix borough typos (brookln → Brooklyn)
  - Clean price ($966 → 966.0)
  - Handle nulls & remove outliers
        ↓
Cleaned CSV (98,109 rows, 21 columns)
        ↓
PostgreSQL (airbnb_db.listings)
        ↓
SQL Analysis — 8 booking questions answered
        ↓
Tableau Dashboard — 6 interactive charts
```

---

## ⚙️ Setup & Installation

### 1. Clone the repository
```bash
git clone https://github.com/your-username/airbnb-nyc-analysis.git
cd airbnb-nyc-analysis
```

### 2. Install Python dependencies
```bash
pip install -r requirements.txt
```

### 3. Download the dataset
Download `Airbnb_Open_Data.csv` from [Kaggle](https://www.kaggle.com/datasets/arianazmoudeh/airbnbopendata) and place it in the `data/` folder.

### 4. Run the cleaning notebook
```bash
jupyter notebook notebooks/Airbnb_EDA_Cleaning.ipynb
```
Run all cells. This produces `Airbnb_Cleaned.csv` and loads data into PostgreSQL.

### 5. Set up PostgreSQL
```sql
CREATE DATABASE airbnb_db;
```
Update credentials in the notebook before running the PostgreSQL cell.

### 6. Run the SQL analysis notebook
```bash
jupyter notebook notebooks/Airbnb_SQL_Analysis.ipynb
```

### 7. Open the Tableau dashboard
Open `dashboard/Airbnb_Dashboard.twbx` in Tableau Public or Tableau Desktop.

---

## 🧹 Data Cleaning Summary

| Issue | Action Taken |
|---|---|
| 541 duplicate rows | Removed, kept first occurrence |
| `license` — 99.99% null | Dropped entirely |
| `house_rules` — 50.8% null | Dropped entirely |
| `country` / `country code` — all "US" | Dropped (zero variance) |
| `NAME` — listing title text | Dropped (not needed for analysis) |
| Borough typos (`brookln`, `manhatan`) | Fixed with `.replace()` |
| Price stored as `$966` string | Stripped `$` and `,`, converted to float |
| Negative `minimum_nights` (down to -1223) | Removed (data entry errors) |
| `availability_365` > 365 | Removed (physically impossible) |
| Non-critical nulls | Filled with median / mode / 0 |

**Raw:** 102,599 rows × 26 columns → **Clean:** 98,109 rows × 21 columns

---

## 🗄️ SQL Analysis — Key Queries & Results

### Q1 — Booking Demand by Borough
```sql
SELECT borough,
       ROUND(AVG(365 - availability_365)::NUMERIC, 1) AS avg_booked_days
FROM listings
GROUP BY borough
ORDER BY avg_booked_days DESC;
```
| Borough | Avg Booked Days |
|---|---|
| Brooklyn | 242.3 |
| Manhattan | 230.6 |
| Queens | 206.5 |
| Bronx | 188.0 |
| Staten Island | 169.0 |

---

### Q2 — Room Type Demand
```sql
SELECT room_type,
       ROUND(AVG(reviews_per_month)::NUMERIC, 2)  AS avg_reviews_per_month,
       ROUND(AVG(365 - availability_365))          AS avg_booked_days
FROM listings
GROUP BY room_type
ORDER BY avg_booked_days DESC;
```
| Room Type | Avg Reviews/Month | Avg Booked Days |
|---|---|---|
| Private room | 1.22 | 234 |
| Entire home/apt | 1.15 | 229 |
| Shared room | 1.11 | 196 |
| Hotel room | 2.87 | 145 |

---

### Q3 — Top 10 Neighbourhoods (min. 20 listings)
```sql
SELECT neighbourhood, borough, COUNT(*) AS listings,
       ROUND(AVG(365 - availability_365)) AS avg_booked_days,
       ROUND(AVG(price))                  AS avg_price
FROM listings
GROUP BY neighbourhood, borough
HAVING COUNT(*) >= 20
ORDER BY avg_booked_days DESC
LIMIT 10;
```
| Neighbourhood | Borough | Listings | Avg Booked Days | Avg Price |
|---|---|---|---|---|
| Navy Yard | Brooklyn | 30 | 296 | $490 |
| Civic Center | Manhattan | 100 | 292 | $621 |
| Downtown Brooklyn | Brooklyn | 169 | 292 | $598 |
| Morningside Heights | Manhattan | 596 | 290 | $632 |
| Stuyvesant Town | Manhattan | 81 | 286 | $684 |
| Woodlawn | Bronx | 29 | 278 | $587 |
| Brooklyn Heights | Brooklyn | 288 | 278 | $595 |
| Nolita | Manhattan | 501 | 268 | $628 |
| Carroll Gardens | Brooklyn | 454 | 267 | $643 |
| Two Bridges | Manhattan | 119 | 267 | $644 |

---

### Q4 — Instant Bookable Impact
```sql
SELECT instant_bookable, COUNT(*) AS total_listings,
       ROUND(AVG(reviews_per_month)::NUMERIC, 2) AS avg_reviews_per_month,
       ROUND(AVG(365 - availability_365))         AS avg_booked_days
FROM listings GROUP BY instant_bookable;
```
| Instant Bookable | Total Listings | Avg Reviews/Month | Avg Booked Days |
|---|---|---|---|
| TRUE | 48,761 | 1.18 | 231 |
| FALSE | 49,348 | 1.18 | 230 |

*Insight: Instant booking has virtually no impact on demand.*

---

### Q5 — Cancellation Policy Impact
```sql
SELECT cancellation_policy,
       ROUND(AVG(reviews_per_month)::NUMERIC, 2) AS avg_reviews,
       ROUND(AVG(365 - availability_365))         AS avg_booked_days
FROM listings GROUP BY cancellation_policy ORDER BY avg_booked_days DESC;
```
| Policy | Avg Reviews | Avg Booked Days |
|---|---|---|
| moderate | 1.19 | 231 |
| flexible | 1.19 | 230 |
| strict | 1.17 | 230 |

*Insight: Policy type has negligible effect on booking demand.*

---

## 📈 Tableau Dashboard

6 interactive sheets assembled into one dashboard:

| Sheet | Chart Type | Insight |
|---|---|---|
| KPI Summary | Text table | 4 headline metrics at a glance |
| Borough Demand | Bar chart | Brooklyn leads demand significantly |
| NYC Map | Geographic scatter | Density clusters in Manhattan & Brooklyn |
| Room Type | Dual-axis bar + line | Private rooms book most, hotels book least |
| Top Neighbourhoods | Horizontal bar | Navy Yard and Civic Center lead |
| Price Distribution | Box plot | All boroughs cluster $400–$900 range |

**Cross-chart filters** via Dashboard Actions — clicking any chart element filters all others simultaneously.

---

## 💡 Booking Demand Methodology

The dataset has no direct "booked" column. Demand is estimated using:
- **`365 - availability_365`** = estimated booked days/year
- **`reviews_per_month`** = correlated proxy (more bookings = more reviews)

This is the same methodology used by AirDNA and published Airbnb research. Hosts can manually block dates, so demand figures are estimates, not exact booking counts.

---

## 📦 requirements.txt

```
pandas>=1.5.0
numpy>=1.23.0
matplotlib>=3.6.0
seaborn>=0.12.0
sqlalchemy>=1.4.0
psycopg2-binary>=2.9.0
jupyter>=1.0.0
```

---

## 👤 Author

**Your Name**
- GitHub: [@your-username](https://github.com/your-username)
- LinkedIn: [your-linkedin](https://linkedin.com/in/your-linkedin)
- Tableau Public: [your-tableau](https://public.tableau.com/profile/your-profile)

---

## 📄 License

Open-source under the [MIT License](LICENSE).

---
*Python • PostgreSQL • Tableau | 2026*
