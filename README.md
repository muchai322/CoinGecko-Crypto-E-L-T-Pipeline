# CoinGecko-Crypto-E-L-T-Pipeline
a data pipeline that extracts data  from an crypto  api to snowflake ware-house


 An end-to-end automated data pipeline that ingests live cryptocurrency market data, transforms it through a multi-layer dbt architecture, and delivers analytics-ready tables in Snowflake — orchestrated by Apache Airflow.

---

## 📌 Overview

This project implements a production-grade ELT (Extract, Load, Transform) pipeline for cryptocurrency market analysis. It collects daily snapshots of the top 750 coins by market cap from the CoinGecko API, loads raw data into Snowflake, and transforms it through a structured dbt project into business-ready analytics tables.

The pipeline runs automatically every morning at 11:00 AM EAT (East Africa Time) and sends email alerts on success or failure.

---

## 🏗️ Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────────────────────────┐
│  CoinGecko API  │────▶│  Apache Airflow  │────▶│             Snowflake               │
│  (750 coins)    │     │  (Orchestration) │     │  ┌─────────────────────────────┐   │
└─────────────────┘     └──────────────────┘     │  │     RAW_CRYPTO_PRICES       │   │
                                                  │  │     (Raw Layer)             │   │
                                                  │  └──────────────┬──────────────┘   │
                                                  │                 │ dbt               │
                                                  │  ┌──────────────▼──────────────┐   │
                                                  │  │   STG_CRYPTO_PRICES         │   │
                                                  │  │   (Staging Layer)           │   │
                                                  │  └──────────────┬──────────────┘   │
                                                  │                 │                   │
                                                  │  ┌──────────────▼──────────────┐   │
                                                  │  │        Marts Layer          │   │
                                                  │  │  • MART_TOP_COINS          │   │
                                                  │  │  • MART_PRICE_MOVERS       │   │
                                                  │  │  • MART_MARKET_DOMINANCE   │   │
                                                  │  └─────────────────────────────┘   │
                                                  └─────────────────────────────────────┘
                                                                    │
                                                  ┌─────────────────▼──────────────────┐
                                                  │         Email Alert (Gmail SMTP)    │
                                                  └─────────────────────────────────────┘
```

---

## ⚙️ Pipeline Tasks

The Airflow DAG runs 4 tasks in sequence:

| Task | Description |
|---|---|
| `extract` | Fetches 750 coins from CoinGecko `/coins/markets` endpoint across 3 paginated requests |
| `load` | Appends raw data into Snowflake with `fetched_at` timestamp for historical tracking |
| `dbt_transform` | Triggers dbt Cloud job via API — runs staging and mart models |
| `notify` | Sends HTML success email via Gmail SMTP |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| **Apache Airflow** (Astro CLI) | Pipeline orchestration and scheduling |
| **Python** | Extract and load logic |
| **CoinGecko API** | Live cryptocurrency market data |
| **Snowflake** | Cloud data warehouse |
| **dbt Cloud** | Data transformation and modeling |
| **Docker** | Containerized Airflow environment |
| **SQLAlchemy** | Snowflake connection and data loading |
| **pandas** | DataFrame handling and column filtering |
| **GitHub** | Version control and CI/CD for dbt models |

---

## 📊 Data Collected

Each pipeline run collects the following fields for 750 coins:

| Column | Description |
|---|---|
| `id` | Unique coin identifier |
| `symbol` | Ticker symbol |
| `name` | Full coin name |
| `current_price` | Price in USD |
| `market_cap` | Total market capitalization |
| `market_cap_rank` | Global rank by market cap |
| `total_volume` | 24hr trading volume |
| `high_24h` / `low_24h` | 24hr price range |
| `price_change_percentage_24h` | 24hr price change % |
| `circulating_supply` | Coins in circulation |
| `last_updated` | CoinGecko last update timestamp |
| `fetched_at` | Pipeline ingestion timestamp |

---

## 🗂️ dbt Project Structure

```
models/
├── staging/
│   ├── sources.yml                  # Snowflake source definition
│   └── stg_crypto_prices.sql        # Clean and standardize raw data
└── marts/
    ├── mart_top_coins.sql            # Top 10 coins by market cap
    ├── mart_price_movers.sql         # Winners and losers (24hr)
    └── mart_market_dominance.sql     # Market share per coin
```

### Staging Layer
Cleans and standardizes raw data:
- Renames columns for clarity
- Casts data types
- Removes bad data (`current_price > 0`)
- Adds derived columns (`price_range_24h`)
- Converts timestamps to East Africa Time (EAT)

### Marts Layer
Answers business questions using the latest snapshot:
- **mart_top_coins** — Top 10 coins by market cap
- **mart_price_movers** — Categorizes coins as Strong Gainer, Gainer, Stable, Loser, Strong Loser
- **mart_market_dominance** — Market share % per coin

---

## ⏱️ Schedule & Configuration

| Setting | Value |
|---|---|
| Schedule | Daily at 11:00 AM EAT (08:00 UTC) |
| Coins per run | 750 |
| Data strategy | Append — full historical snapshots preserved |
| Retries | 3 per task |
| Email alerts | Success and failure notifications |

---

## 🗂️ Project Structure

```
airflow-projects/
├── dags/
│   └── congeck_api/
│       ├── dag.py                   # Airflow DAG — TaskFlow API
│       ├── extract_api.py           # CoinGecko fetch logic
│       ├── load_to_snowflake.py     # Snowflake load logic
│       └── __init__.py
├── .env                             # Environment variables (not committed)
├── requirements.txt                 # Python dependencies
├── Dockerfile                       # Astro Runtime image
└── README.md
```

---



### Prerequisites
- [Astro CLI](https://docs.astronomer.io/astro/cli/install-cli)
- Docker
- Snowflake account
- CoinGecko API access (free tier)
- dbt Cloud account
- Gmail account with App Password enabled

### Setup

**1. Clone the repo**
```bash
git clone https://github.com/Muchai322/coingecko-elt-pipeline.git
cd coingecko-elt-pipeline
```

**2. Create your `.env` file**
```bash
# Snowflake
SNOWFLAKE_USER=your_user
SNOWFLAKE_PASSWORD=your_password
SNOWFLAKE_ACCOUNT=your_account
SNOWFLAKE_WAREHOUSE=your_warehouse
SNOWFLAKE_DATABASE=your_database
SNOWFLAKE_SCHEMA=your_schema

# dbt Cloud
DBT_API_TOKEN=your_dbt_service_token

# Gmail SMTP
AIRFLOW__SMTP__SMTP_HOST=smtp.gmail.com
AIRFLOW__SMTP__SMTP_PORT=587
AIRFLOW__SMTP__SMTP_STARTTLS=True
AIRFLOW__SMTP__SMTP_SSL=False
AIRFLOW__SMTP__SMTP_USER=your_email@gmail.com
AIRFLOW__SMTP__SMTP_PASSWORD=your_app_password
AIRFLOW__SMTP__SMTP_MAIL_FROM=your_email@gmail.com
```

**3. Start Airflow**
```bash
astro dev start
```

**4. Open the Airflow UI**
```
http://localhost:8080
```

Trigger the `coingecko_api` DAG manually or wait for the scheduled run.

---

## 📈 Sample Queries

```sql
-- Top 10 coins right now
SELECT * FROM DBTDEMO.DBT.MART_TOP_COINS;

-- Biggest winners and losers today
SELECT * FROM DBTDEMO.DBT.MART_PRICE_MOVERS
WHERE price_movement_category IN ('Strong Gainer', 'Strong Loser');

-- Bitcoin market dominance over time
SELECT coin_name, market_dominance_pct, fetched_at_eat
FROM DBTDEMO.DBT.MART_MARKET_DOMINANCE
WHERE coin_id = 'bitcoin'
ORDER BY fetched_at_eat;

-- Track Bitcoin price history
SELECT coin_name, current_price_usd, fetched_at_eat
FROM DBTDEMO.DBT.STG_CRYPTO_PRICES
WHERE coin_id = 'bitcoin'
ORDER BY fetched_at_eat;
```

---

## 🔄 How It Works

1. **Every morning at 11am EAT** Airflow wakes up and triggers the pipeline
2. **Extract** — Python fetches 750 coins from CoinGecko across 3 API calls with rate limiting
3. **Load** — Data is appended to `RAW_CRYPTO_PRICES` in Snowflake with a timestamp
4. **dbt Transform** — Airflow triggers a dbt Cloud job via API which runs staging and mart models
5. **Notify** — A success email is sent confirming the run completed
6. If any task fails, a failure alert is sent automatically

---

## 👤 Author

**Joseph Muchai Ndungu**
 Data Engineer & analytics engineer

  [GitHub](https://github.com/Muchai322)
