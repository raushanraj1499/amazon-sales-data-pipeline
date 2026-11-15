# Amazon Sales ETL Pipeline with PySpark and Airflow

An end-to-end data engineering project that downloads an Amazon sales dataset, processes it with PySpark using a Bronze-Silver-Gold architecture, orchestrates the workflow with Apache Airflow, and loads analytics-ready tables into PostgreSQL. The complete development environment runs with Docker Compose.

Built by [Raushan Raj](https://github.com/raushanraj1499).

## Architecture

![Amazon sales ETL architecture](./Architecture%20Diagram.png)

The pipeline has three stages:

1. **Bronze / Extract** - downloads the public Amazon Sales Dataset from Kaggle and stores the raw CSV.
2. **Silver / Transform** - cleans prices, discounts, ratings, and review counts; derives rating and discount buckets; and writes Parquet output.
3. **Gold / Load** - builds business-level aggregations, saves them as Parquet, and loads them into PostgreSQL.

Airflow coordinates the table setup and all three PySpark jobs.

## Tech stack

- Python 3
- Apache Airflow 2.8.1
- Apache Spark / PySpark 3.4.1
- PostgreSQL 13
- Docker and Docker Compose
- KaggleHub

## Project structure

```text
.
├── .github/workflows/ci.yml       # Python syntax check
├── dags/amazon_sales_etl.py       # Airflow DAG
├── data/                           # Runtime data (ignored by Git)
├── spark/jobs/
│   ├── extract.py                  # Bronze extraction
│   ├── transform.py                # Silver transformations
│   └── gold_load.py                # Gold aggregations and database load
├── sql/                            # PostgreSQL table definitions
├── Architecture Diagram.png
├── docker-compose.yaml
├── Dockerfile
├── practice.ipynb                  # Exploratory notebook
└── requirements.txt
```

## Gold tables

| Table | Purpose |
| --- | --- |
| `product_summary` | Product-level price, rating, discount, and engagement metrics |
| `category_summary` | Category-level product, rating, discount, and price metrics |
| `top_products` | Top five products in each category by engagement and rating strength |
| `discount_effectiveness` | Rating and engagement performance by discount bucket |

The table-creation tasks truncate existing result tables before each run, so manually rerunning the DAG does not duplicate gold records.

## Run locally

### Prerequisites

- Docker Desktop with Docker Compose
- At least 4 GB of memory available to Docker
- Internet access on the first run to download container images, Python packages, the dataset, and the PostgreSQL JDBC driver

### Start the pipeline services

```bash
docker compose up --build
```

The initialization container migrates the Airflow database, creates the local Airflow user, and makes the PostgreSQL connection available to the DAG automatically.

Open [http://localhost:8080](http://localhost:8080), sign in, and trigger `amazon_sales_etl`.

- Username: `airflow`
- Password: `airflow`

These credentials are intended only for local development. Do not use them in a public or production deployment.

To stop the services:

```bash
docker compose down
```

To also remove the local PostgreSQL volume:

```bash
docker compose down --volumes
```

## Dataset

The extraction job downloads the public [Amazon Sales Dataset on Kaggle](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset). Generated raw data, Parquet outputs, caches, and Airflow logs are intentionally excluded from Git.

## Business questions

- Which product categories perform best by rating, reviews, and pricing?
- How do discount levels relate to ratings and customer engagement?
- Which products have the strongest combination of rating and review volume?
- Which curated tables can support BI dashboards and downstream analysis?

## Possible improvements

- Add automated data-quality checks and alerting
- Add unit and integration tests for transformations
- Support incremental loads and change-data capture
- Store data in cloud object storage such as Amazon S3
- Connect a BI tool such as Power BI or Apache Superset

## Notes

- The project is configured for local demonstration and learning.
- PostgreSQL and Airflow use local development credentials from `docker-compose.yaml`.
- The pipeline downloads its input automatically; no dataset needs to be committed to the repository.
