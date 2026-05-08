<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=rounded&height=250&color=gradient&text=Ashley%20Esau-nl-Data%20Professional&fontColor=000000" width="50%" />
</p>

---

## About Me

Hi, I am **Ashley Esau**, a data professional who builds production-style data systems.

I started in market research data processing, which taught me early that data quality is not a technical concern. It is a business one. That foundation has shaped everything since.

From there I grew curious about how larger systems work. How data moves from raw inputs to decisions. How companies like Netflix or Spotify make sense of the behaviour happening inside their products. How the infrastructure underneath an analytics layer actually gets built. Each project I take on starts with a question I cannot shake, and ends with a system I understand deeply.

I have worked across retail pipelines, fintech analytics, cloud-scale dimensional models, and subscription economics. The domain changes. The discipline does not.

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

**Warehouses and Databases**
- DuckDB
- Google BigQuery
- AWS Redshift
- AWS Athena

**Orchestration**
- Apache Airflow

**Data Quality**
- Soda Core

**Cloud Infrastructure**
- AWS (S3, Glue, Athena, Redshift)
- Google Cloud (BigQuery, Cloud Storage)

**Dashboarding and Visualisation**
- Streamlit
- Plotly
- Looker Studio
- Metabase

**Other**
- Docker
- Git

---

# Featured Projects

---

## 1. SaaS Analytics Warehouse

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
- Two-institution design: Institution A (control, real Plaid Sandbox data) and Institution B (synthetic seed with engineered behavioral contrast: irregular income, high merchant concentration, recurring negative cashflow. This was a deliberate design decision to produce analytically meaningful risk signal.
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

## 3. Marketing Analytics Warehouse

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

## 4. Fintech Portfolio Analytics

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

## 5. Retail Data Pipeline with Modern Data Stack

**Repository:** <a href="https://github.com/ashleyesau/airflow_project" target="_blank">github.com/ashleyesau/airflow_project</a>

A fully orchestrated ELT pipeline processing over 540,000 retail transactions.

### Stack

- Apache Airflow
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

## 6. NYC Parking Tickets Dimensional Model

**Repository:** <a href="https://github.com/ashleyesau/parking_tickets" target="_blank">github.com/ashleyesau/parking_tickets</a>

A large-scale dimensional modeling project built on 42 million public records using AWS and dbt.

### Stack

- AWS S3
- AWS Glue
- AWS Athena
- AWS Redshift
- dbt

### What I Built

- Reduced and validated the raw dataset
- Transparent invalid record tracking instead of silent deletion
- Star schema with `fact_tickets`, `dim_date`, `dim_vehicle`, `dim_violation`, and `dim_officer`
- Data validation queries for duplicates, date inconsistencies, and anomalies
- Structured modeling optimized for scalable analytics

### What It Demonstrates

- Handling high-volume datasets
- Schema design under ambiguity
- Strong dimensional modeling discipline
- Data quality transparency

This was my foundational engineering project where I learned to think structurally about data systems.

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
