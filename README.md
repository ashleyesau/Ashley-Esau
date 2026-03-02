<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=rounded&height=250&color=gradient&text=Ashley%20Esau-nl-Data%20Professional&fontColor=000000" width="50%" />
</p>

---

## 👋 About Me

Hi, I am **Ashley Esau**, a data professional focused on building reliable, production-style data systems.

I started my career in market research data processing, where I learned that data quality directly impacts business decisions. That foundation shaped how I approach engineering today: structured, quality-first, and business-aware.

Over the past few years, I have transitioned into the modern data stack, building dimensional models, orchestrated pipelines, automated quality frameworks, and interactive analytics layers.

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

## 1️⃣ Fintech Portfolio Analytics

🔗 **Repository:** https://github.com/ashleyesau/fintech_modeling  

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

## 2️⃣ Retail Data Pipeline with Modern Data Stack

🔗 **Repository:** https://github.com/ashleyesau/airflow_project  

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

## 3️⃣ NYC Parking Tickets Dimensional Model

🔗 **Repository:** https://github.com/ashleyesau/parking_tickets  

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

LinkedIn: https://linkedin.com/in/ashley-esau  
GitHub: https://github.com/ashleyesau  
Email: ashley.esau@gmail.com  

Open to conversations about data engineering, fintech platforms, and building resilient data systems.
