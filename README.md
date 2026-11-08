<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=rounded&height=250&color=gradient&text=Ashley%20Esau-nl-Data%20Professional&fontColor=000000" width="50%" />
</p>

---

## About Me

Hi, I am **Ashley Esau**, a data professional who builds production-style data systems.

I started in market research data processing, which taught me early that data quality is not a technical concern. It is a business one. That foundation has shaped everything since.

From there I grew curious about how larger systems work. How data moves from raw inputs to decisions. How companies like Netflix or Spotify make sense of the behaviour happening inside their products. How the infrastructure underneath an analytics layer actually gets built. Each project I take on starts with a question I cannot shake, and ends with a system I understand deeply.

I have worked across retail pipelines, fintech analytics, change data capture, warehouse modelling on BigQuery, and subscription economics. The domain changes. The discipline does not.

I care about:

- Clean, scalable data models
- Automated validation and testing
- Infrastructure that fails loudly instead of silently
- Engineering that supports real financial decisions

---

## Core Stack

**Languages**
- Python
- SQL

**Transformation**
- dbt Core (DuckDB and BigQuery adapters)
- dbt-utils

**Warehouses, Databases and Table Formats**
- Google BigQuery
- BigQuery ML
- PostgreSQL (logical replication)
- Apache Iceberg
- DuckDB

**Orchestration**
- Apache Airflow
- Dataform

**Data Quality and Testing**
- Soda Core
- dbt tests
- pytest

**Cloud and Infrastructure as Code**
- Google Cloud Platform (BigQuery, BigQuery ML, Cloud Storage, Dataform, IAM)
- Terraform

**Dashboarding and Visualisation**
- Lightdash
- Streamlit
- Plotly
- Looker Studio
- Metabase

**Other**
- Docker
- Git and GitHub Actions

---

# Featured Projects

---

## 1. Fintech Payments Platform

**Repository:** <a href="https://github.com/ashleyesau/fintech-payments" target="_blank">github.com/ashleyesau/fintech-payments</a>

An end-to-end data pipeline for a South African SME payments and lending business, starting at a Postgres source. A custom change data capture consumer reads the write-ahead log and lands files in Cloud Storage, a Bronze writer turns them into Apache Iceberg tables, BigQuery and dbt build the marts, Lightdash serves the dashboards, and a five-task Airflow DAG chains the steps together. The largest run pushed 8.5 million events through it, and the marts show just over R1 billion in simulated gross payment value. The source data is synthetic. The pipeline is the point.

### Overview

Many small South African businesses have no payslips and no credit history, but most now have a card machine. Every transaction is a trail left behind by something real: how often the shop opens, how it recovers after a slow month. This project follows that trail from the counter to the lender, and builds the infrastructure that catches each transaction, moves it, checks it and turns it into something a lender can read.

The marts cover gross payment value, settlement reconciliation, merchant health, and loan portfolio quality including portfolio at risk and vintage curves.

### Architecture

```
PostgreSQL 16 (wal_level=logical)
        |
Custom CDC consumer
(pgoutput decoded from a logical replication slot)
        |
Google Cloud Storage
(gzipped NDJSON landing, idempotent, replayable)
        |
Bronze writer (PyIceberg)
(5 Apache Iceberg tables on GCS, partitioned by month)
        |
BigQuery external tables
        |
dbt: staging, intermediate, marts
(27 models, 165 schema tests)
        |
Lightdash
(8 dashboards, 32 charts)

Airflow DAG (manual trigger):
bronze_write > bronze_verify > register_bq > dbt_marts > dbt_test
```

### Stack

| Layer | Technology |
| --- | --- |
| Source | PostgreSQL 16 with logical replication |
| Change capture | Custom Python consumer decoding pgoutput |
| Landing | Google Cloud Storage (gzipped NDJSON) |
| Bronze | Apache Iceberg via PyIceberg |
| Warehouse | BigQuery (external tables over Iceberg) |
| Transformations | dbt Core (BigQuery adapter) |
| Orchestration | Apache Airflow 2.9.3 (LocalExecutor, Docker Compose) |
| BI | Lightdash Cloud |
| Infrastructure | Terraform (buckets, service accounts, IAM, datasets) |
| CI | GitHub Actions (pytest, dbt deps, terraform validate) |

### Engineering Highlights

- Reads changes from the Postgres write-ahead log with SQL-level peek and advance calls on a logical replication slot, rather than the streaming replication protocol. This keeps the consumer simple and needs no replication role.
- Memory-bounded Iceberg writer built around PyIceberg limits: a two-pass reader with an index of about 12 bytes per event and 100,000-row batches. Peak memory fell from 5.0 GB to roughly 670 MB on an 8.5 million event load. Crash recovery and idempotency are covered by a test suite.
- Month partitioning across five Bronze tables: 60 data files a year instead of 1,825 with day partitioning, which cuts storage API operations roughly 14 times.
- A contract-validation step (`bronze.check`) between the Bronze write and BigQuery registration, so a malformed batch stops the pipeline before dbt sees it.
- A Type 2 slowly changing dimension for merchants, built with LAG window functions over daily snapshots, because a dbt snapshot could not build the correct history. It produces 4,991 history rows for 1,000 merchants.
- A 30-day rolling payment value model per merchant, rebuilt as a table with an integer day axis after BigQuery materialized views rejected the interval window.
- Diagnosed a replication slot invalidation during a large backfill (about 11 GB of log against a 3 GB cap) and resized the retention limit from measurement instead of an estimate.
- Airflow 2.9 and PyIceberg need incompatible SQLAlchemy versions, so the pipeline runs in an isolated virtualenv that each task calls directly.
- Least-privilege infrastructure in Terraform: a dedicated pipeline-runner service account, and a custom IAM role holding only the single BigQuery connection permission that external table creation needs.
- 165 dbt schema tests, run by the DAG, plus a pytest suite and a GitHub Actions workflow that runs Postgres 16 with logical decoding.

### Known Limits

Documented in the repo, not hidden:

- The DAG is triggered manually. There is no schedule, no retries and no alerting.
- The dbt tests run after the marts are built, so a failure stops the run but does not stop bad data reaching Lightdash. Testing each layer before building the next is the fix.
- The impersonated credentials depend on my own login session, so the pipeline is not unattended yet.
- CI resolves dbt packages but does not build or test the models against BigQuery.
- The Iceberg catalog is a local SQLite file rebuilt from a marker in Cloud Storage. That suits one machine and would not survive more.

---

## 2. Fintech Behavioral Analytics Platform

**Repository:** <a href="https://github.com/ashleyesau/fintech-behavioral-analytics" target="_blank">github.com/ashleyesau/fintech-behavioral-analytics</a>

A full-stack analytics engineering project simulating the internal data platform of a fintech company. It covers the entire pipeline: live API ingestion from the Plaid Sandbox, cloud storage on GCS, BigQuery warehousing, a ten-model layered dbt transformation architecture, 81 automated tests, and daily orchestration via Apache Airflow running in Docker.

### Overview

The project was built around a question I kept returning to while working through the modern data stack: what does it actually look like when all of these pieces connect? Not in a tutorial where the data is already clean, but in practice, where things break in ways you did not anticipate and the debugging teaches you more than the building did.

The platform tracks customer financial health across two institutions and surfaces behavioral risk signals in transaction data. The analytical output lives in `mart_risk_signals`, which identifies customers combining high merchant concentration (top three merchants exceeding 60% of spend) with two or more consecutive months of negative net cash flow. These are the highest-priority intervention targets for a risk team.

### Architecture

```
Plaid Sandbox API (2 institutions)
        |
Python Extraction Layer
(cursor-based sync, retry logic, GCS write)
        |
Google Cloud Storage
(partitioned by institution + date, immutable raw JSON)
        |
BigQuery Raw Tables (Bronze)
        |
dbt Staging Models (Silver)
        |
dbt Intermediate Models
        |
dbt Mart Models (Gold)
        |
Apache Airflow DAG
(daily orchestration with quality gates between layers)
```

Each layer reads only from the layer directly above it. When something breaks, you know exactly which layer owns the problem.

### Stack

| Layer | Technology |
| --- | --- |
| API Source | Plaid Sandbox API |
| Extraction | Python 3.13 |
| Raw Storage | Google Cloud Storage (partitioned by institution + date) |
| Data Warehouse | BigQuery (us-central1) |
| Transformations | dbt Core (BigQuery adapter) |
| Orchestration | Apache Airflow 2.9.1 (LocalExecutor) |
| Container Runtime | Docker Compose |

### Engineering Highlights

- Cursor-based sync via `/transactions/sync` rather than the deprecated date-based endpoint. The cursor advances only after a successful GCS write, guaranteeing no silent transaction loss on write failures.
- Two-institution design: Institution A (control, real Plaid Sandbox data) and Institution B (synthetic seed with engineered behavioral contrast: irregular income, high merchant concentration, recurring negative cashflow). This was a deliberate design decision to produce analytically meaningful risk signal.
- Cross-join spine pattern in `int_account_monthly_cashflow` to preserve zero-transaction months, making consecutive cashflow streak calculations correct rather than misleading
- Gaps-and-islands pattern in `int_customer_risk_signals` for maximum consecutive negative cashflow streak per account
- Merchant concentration ratio derived from top three merchants by debit spend, combined with cashflow streak into a single `combined_risk_flag`
- Four mart models serving distinct stakeholder perspectives: risk team, finance team, product team, and data team
- 81 dbt tests across 10 models covering uniqueness, not-null constraints, accepted values, and custom business logic
- 7-task Airflow DAG with a quality gate between staging and intermediate: staging tests must pass before any intermediate model runs
- LocalExecutor selected over CeleryExecutor after CeleryExecutor's six-container stack exceeded 4GB RAM and killed tasks under memory pressure. A real infrastructure constraint with a real solution.

### Data Quality Bugs Found and Fixed

Three silent data loss bugs were discovered during the build, none of them obvious.

**The load loop bug.** The BigQuery loader used `WRITE_TRUNCATE` on date partitions with institutions as the outer loop. Institution B's data overwrote institution A's on every write. The entire first institution was absent from every downstream model. Fix: swap loop order so all institutions for a given data type are written in a single operation.

**The pending filter bug.** Plaid Sandbox returns `pending = NULL` for one institution rather than explicit `false`. A naive `WHERE pending = false` filter silently excluded every institution A transaction because `NULL = false` evaluates to NULL, not true. Fix: `WHERE COALESCE(pending, false) = false`.

**The name collision bug.** A CTE named `merchant_spend` contained a column also named `merchant_spend`. When a downstream CTE referenced `ORDER BY merchant_spend`, BigQuery resolved the name to the CTE itself (a STRUCT) and rejected the sort. The error message pointed at the expression type, not the naming collision. Fix: rename CTE to `merchant_spend_by_name`, column to `spend_amount`. Now a documented project convention.

---

## 3. Retail Medallion Pipeline on BigQuery

**Repository:** <a href="https://github.com/ashleyesau/retail-project" target="_blank">github.com/ashleyesau/retail-project</a>

A Bronze-Silver-Gold pipeline in BigQuery that takes 10,000 raw retail transactions through profiling, cleaning, and customer segmentation with BigQuery ML, orchestrated with Dataform.

### Stack

- Google BigQuery
- BigQuery ML (k-means)
- Dataform
- SQL

### Architecture

Raw CSV → Bronze (all-STRING schema) → Silver (cleaned and typed) → Gold (BQML k-means segments)

### Engineering Highlights

- Profiled the Bronze table before writing any transformation, so every cleaning rule is backed by evidence
- Found nulls stored as the literal string `'NULL'`, which a plain `COALESCE` walks straight past. Fixed with `NULLIF` before `COALESCE`
- Cross-tabbed 407 negative amounts against `is_returned` and found 282 were not refunds, so they were removed as corrupt records (10,000 rows in, 9,593 out)
- Identified that `signup_date` was generated per transaction rather than per customer, and documented the limitation in the SQL instead of hiding it
- Trained a six-cluster k-means model with standardised features, and wrote the cluster assignments back to an analytics table
- Dataform assertions (non-null and row conditions) enforcing quality in the Silver layer

### What It Demonstrates

- Designing each layer to handle one class of data problem
- Making cleaning decisions from evidence
- Being honest about what a model can and cannot show on uniform data

---

## 4. Retail Data Pipeline with Modern Data Stack

**Repository:** <a href="https://github.com/ashleyesau/airflow_project" target="_blank">github.com/ashleyesau/airflow_project</a>

A fully orchestrated ELT pipeline processing over 540,000 retail transactions.

### Stack

- Apache Airflow
- Google Cloud Storage
- Google BigQuery
- dbt
- Soda Core
- Docker
- Metabase

### What I Built

- Orchestrated ingestion to transformation to quality to reporting
- Star schema with three dimension tables and one fact table
- More than 20 automated data quality checks
- Surrogate key generation for non-unique product variations
- Intelligent handling of 27 percent null customer IDs through segmentation
- Containerized environment for reproducibility

### What It Demonstrates

- Handling messy real-world data
- Production-grade orchestration
- Multi-layer quality gates at source, transform, and report levels
- Cloud-native analytics architecture

This project demonstrates the ability to design and automate complete analytics systems, not just write SQL.

---

## 5. SaaS Analytics Warehouse

**Repository:** <a href="https://github.com/ashleyesau/saas_modeling" target="_blank">github.com/ashleyesau/saas_modeling</a>

**Live Dashboard:** <a href="https://saasmodeling-klogjaq7u6jsgczkzsdtwp.streamlit.app/" target="_blank">saasmodeling-klogjaq7u6jsgczkzsdtwp.streamlit.app</a>

A full subscription lifecycle analytics system built with DuckDB, dbt, and Streamlit.

### Overview

This project models the subscription economics of a fictional SaaS company across a two-year period. It answers the questions that matter most to subscription businesses:

- How is revenue moving, and why?
- Which accounts are at risk of churning?
- What signals appear before a customer leaves?
- Which acquisition channels produce the highest lifetime value?
- How do cohorts retain over time, by plan tier?

### Architecture

Raw CSV → Staging in dbt → Grain Conflict Resolution → Mart Layer → Interactive Streamlit Dashboard

### Engineering Highlights

- Three-layer dbt architecture: staging, intermediate, and marts
- Grain conflict resolution across up to 17 subscription rows per account-month, resolved via a deterministic selection rule codified in `int_subscription_month_selected`
- Full MRR revenue bridge with new, expansion, contraction, churn, and reactivation components
- 90-day rolling feature breadth window powering a four-segment risk model in `fct_account_health`
- Cohort retention model with logo retention and NRR by plan tier in `fct_cohort_retention`
- Pre-churn pattern classification derived from MRR movement signals in `fct_churn`
- 40 dbt tests across six models, all passing with zero warnings
- Four-page Streamlit dashboard with custom sidebar navigation, shared utils module, and Plotly charts throughout

### Key Findings

- Accounts using 10 or more distinct features in 90 days have near-zero churn. Below that threshold, churn is flat at 8-12% regardless of tier or volume.
- 435 medium-risk accounts hold $1.6M MRR, the highest-leverage Customer Success intervention target.
- Partner is the highest-LTV acquisition channel despite having the smallest volume.
- 19.2% of all churn events follow an upgrade. Overcommit and unmet expectations are the dominant exit pattern.

This project demonstrates what it takes to build the foundation that makes subscription analytics honest, not just possible.

---

## 6. Marketing Analytics Warehouse

**Repository:** <a href="https://github.com/ashleyesau/marketing_analysis" target="_blank">github.com/ashleyesau/marketing_analysis</a>

**Live Dashboard:** <a href="https://marketinganalysis-nxmm6e4ytzj8mxcfjxkm5f.streamlit.app/" target="_blank">marketinganalysis-nxmm6e4ytzj8mxcfjxkm5f.streamlit.app</a>

An end-to-end analytics engineering project modeling customer segmentation, funnel analysis, product performance, and cohort retention on a synthetic e-commerce dataset of 2 million events.

### Overview

This project was built to close the gap between knowing how dbt works and demonstrating that you can build a production-quality layered pipeline: one where data quality issues surface mid-project, modeling decisions compound across layers, and a metric that looks correct can be silently wrong.

It answers real business questions around:

- Which traffic sources convert, and by how much?
- How do customer cohorts retain over time?
- Which products and categories drive revenue?
- What does campaign attribution actually look like when the data is messy?
- What do A/B experiment results tell us about purchase behaviour?

### Architecture

Raw CSVs → dbt seed → Staging → Intermediate → Mart Layer → Five-page Streamlit Dashboard

### Engineering Highlights

- Three-layer dbt pipeline across five source tables: customers, transactions, events, products, and campaigns
- 10,449 null-revenue transactions excluded after EDA confirmed they were unresolvable at source. Not silently dropped, but documented.
- Funnel analysis page replaced after identifying a synthetic data artifact producing near-identical counts across all segment dimensions. The events table variance became the foundation instead.
- Column naming conflict between customer acquisition channel and session-level traffic source resolved explicitly rather than papered over
- Ten mart models covering dimensions and pre-aggregated fact tables consumed directly by the dashboard
- Five-page Streamlit dashboard covering customer segmentation, traffic and conversion, product performance, campaign attribution, and A/B experiment results

### Key Findings

- Email converts at 9.8% and paid search at 9.3%, roughly four times the rate of organic and direct traffic at 2.3%
- Variant B in the A/B experiment reaches a 12.2% purchase rate versus 9.0% for control
- Electronics leads revenue at $3.55M, with a consistent November and December spike every year
- 36% of customers have tracked events but have never completed a transaction

---

## 7. Fintech Portfolio Analytics

**Repository:** <a href="https://github.com/ashleyesau/fintech_modeling" target="_blank">github.com/ashleyesau/fintech_modeling</a>

**Live Dashboard:** <a href="https://fintechmodeling-pybu29dx7xvnq3um9fjmnu.streamlit.app/" target="_blank">fintechmodeling-pybu29dx7xvnq3um9fjmnu.streamlit.app</a>

A production-style end-to-end fintech analytics system built with dbt, DuckDB, and Streamlit.

### Overview

This project simulates a fintech company managing savings, checking, credit, and investment products. It answers real business questions around:

- Wealth concentration
- Customer risk segmentation
- Loan-to-income leverage
- Balance distribution using decile analysis
- Gini coefficient and Lorenz curve modeling

### Architecture

Raw CSV → Staging in dbt → Star Schema → Intermediate Metrics → KPI Layer → Interactive Streamlit Dashboard

### Engineering Highlights

- Star schema design with `dim_customer`, `dim_account`, and `fct_customer_account_snapshot`
- Intermediate semantic layer `int_customer_account_metrics`
- Pre-aggregated KPI models for wealth concentration, leverage, and deciles
- Advanced window functions such as NTILE and ROW_NUMBER
- Automated dbt tests for uniqueness, relationships, and numeric ranges
- In-app Gini coefficient calculation
- Dynamic Streamlit dashboard querying DuckDB directly

This project mirrors how real fintech data teams structure analytics infrastructure.

---

## Engineering Philosophy

I am drawn to working on the systems others can depend on without thinking about them. When the infrastructure works, it is invisible - analysts explore trends instead of fixing broken joins, dashboards are trusted instead of questioned, decisions get made on time. The only moment infrastructure gets attention is when it breaks. That is the responsibility I have learned to embrace: building systems so reliable that part of their success is measured by how invisible they remain.

I believe in:

- Designing with intention
- Making assumptions explicit
- Automating validation
- Documenting trade-offs
- Building systems that are durable

I am particularly interested in fintech and data platform roles where data quality directly impacts financial decisions.

---

## Connect

<a href="https://linkedin.com/in/ashley-esau" target="_blank">LinkedIn</a> |
<a href="https://github.com/ashleyesau" target="_blank">GitHub</a> |
<a href="mailto:ashley.esau@gmail.com">ashley.esau@gmail.com</a>

Open to conversations about data engineering, fintech platforms, and building resilient data systems.
