<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=rounded&height=250&color=gradient&text=Ashley%20Esau-nl-Data%20Professional&fontColor=000000" width="50%" />
</p>

---

## 👋 About Me

Hi, I am **Ashley Esau**, a data professional who builds production-style data systems.

I started in market research data processing, which taught me early that data quality is not a technical concern — it is a business one. That foundation has shaped everything since.

From there I grew curious about how larger systems work. How data moves from raw inputs to decisions. How companies like Netflix or Spotify make sense of the behaviour happening inside their products. How the infrastructure underneath an analytics layer actually gets built. Each project I take on starts with a question I cannot shake, and ends with a system I understand deeply.

I have worked across retail pipelines, fintech analytics, cloud-scale dimensional models, and subscription economics. The domain changes. The discipline does not.

I care about:

- Clean, scalable data models
- Automated validation and testing
- Infrastructure that fails loudly instead of silently
- Engineering that supports real financial decisions

---

## 🛠 Core Stack

- SQL for advanced transformations and dimensional modeling
- dbt for semantic modeling, testing, and documentation
- Apache Airflow for orchestration and DAG design
- DuckDB, BigQuery, and Postgres
- Python
- Docker
- Git
- Soda Core for automated data quality checks

---

# 🚀 Featured Projects

---

## 1️⃣ SaaS Analytics Warehouse

🔗 **Repository:** <a href="https://github.com/ashleyesau/saas_modeling" target="_blank">github.com/ashleyesau/saas_modeling</a>
📊 **Live Dashboard:** <a href="https://saasmodeling-klogjaq7u6jsgczkzsdtwp.streamlit.app/" target="_blank">saasmodeling-klogjaq7u6jsgczkzsdtwp.streamlit.app</a>

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
- 435 medium-risk accounts hold $1.6M MRR — the highest-leverage Customer Success intervention target.
- Partner is the highest-LTV acquisition channel despite having the smallest volume.
- 19.2% of all churn events follow an upgrade — overcommit and unmet expectations are the dominant exit pattern.

This project demonstrates what it takes to build the foundation that makes subscription analytics honest, not just possible.

---

## 2️⃣ Fintech Portfolio Analytics

🔗 **Repository:** <a href="https://github.com/ashleyesau/fintech_modeling" target="_blank">github.com/ashleyesau/fintech_modeling</a>

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

## 3️⃣ Retail Data Pipeline with Modern Data Stack

🔗 **Repository:** <a href="https://github.com/ashleyesau/airflow_project" target="_blank">github.com/ashleyesau/airflow_project</a>

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

## 4️⃣ NYC Parking Tickets Dimensional Model

🔗 **Repository:** <a href="https://github.com/ashleyesau/parking_tickets" target="_blank">github.com/ashleyesau/parking_tickets</a>

A large-scale dimensional modeling project built on 42 million public records using AWS and dbt.

### Stack

- AWS S3
- AWS Glue
- Redshift
- dbt

### What I Built

- Reduced and validated the raw dataset
- Transparent invalid record tracking instead of silent deletion
- Star schema with:
  - `fact_tickets`
  - `dim_date`
  - `dim_vehicle`
  - `dim_violation`
  - `dim_officer`
- Data validation queries for duplicates, date inconsistencies, and anomalies
- Structured modeling optimized for scalable analytics

### What It Demonstrates

- Handling high-volume datasets
- Schema design under ambiguity
- Strong dimensional modeling discipline
- Data quality transparency

This was my foundational engineering project where I learned to think structurally about data systems.

---

## 🧠 Engineering Philosophy

"Good data engineering is quiet infrastructure. When it works, nobody notices, but everything depends on it."

I believe in:

- Designing with intention
- Making assumptions explicit
- Automating validation
- Documenting trade-offs
- Building systems that are durable

I am particularly interested in fintech and data platform roles where data quality directly impacts financial decisions.

---

## 📬 Connect

<a href="https://linkedin.com/in/ashley-esau" target="_blank">LinkedIn</a> |
<a href="https://github.com/ashleyesau" target="_blank">GitHub</a> |
<a href="mailto:ashley.esau@gmail.com">ashley.esau@gmail.com</a>

Open to conversations about data engineering, fintech platforms, and building resilient data systems.
