# 🎓 Canadian Higher Ed: Regional Specialization Dashboard

**An End-to-End Data Stack for Analyzing University Program Specialization**

## 📖 Project Overview

This project identifies regional talent specializations across Canadian provinces by calculating **Location Quotients (LQ)**. It transforms raw Statistics Canada (PSIS) data into an interactive dashboard, helping University Administrators answer: *"In which fields does our province have a competitive talent advantage compared to the national average?"*

### Key Features

* **Data Engineering:** `dlt` ingests 3M+ rows of StatCan PSIS data directly into PostgreSQL.
* **Analytics Engineering:** `dbt` transforms raw data into a dimensional model, handling complex provincial/national aggregations and data cleansing.
* **Location Quotient Logic:** Custom SQL models identify specializations where LQ > 1.2 (indicating a significant regional focus).
* **Interactive UI (Dashboard):** A PHP-powered dashboard using Chart.js to visualize specializations with dynamic updates.

---

## 🏗 System Architecture

The project follows a modern data stack architecture, containerized and deployed on a single DigitalOcean Droplet via Dokku.

1. **Ingestion (`/data-pipeline`):** Python script using `dlt` to load CSV/Parquet data.
2. **Transformation (`/data-pipeline/dbt_transform`):** dbt models for cleaning, deduplication (filtering "Total" rows), and LQ calculations.
3. **Storage:** PostgreSQL managed by the `dokku-postgres` plugin.
4. **Frontend (`/dashboard`):** PHP API endpoints serving JSON to a Chart.js frontend.

---

## 📂 Repository Structure

```text
.
├── dashboard/               # PHP Web Application
│   ├── api/                 # PHP Data API (get_lq_data.php)
│   ├── index.php            # Main Dashboard UI
│   └── assets/              # JS (Chart.js) and CSS
├── data-pipeline/           # Data Engineering Logic
│   ├── dlt_loader.py        # dlt Ingestion Script
│   ├── dbt_transform/       # dbt Project (Models, Macros, Tests)
│   ├── Dockerfile           # Environment definition for the worker
│   └── requirements.txt     # Python dependencies
└── README.md

```

---

## 🚀 Deployment (Dokku)

This project is optimized for a **monorepo deployment** on a single server.

### 1. Database Setup

```bash
dokku postgres:create psis-db

```

### 2. App Orchestration

We deploy two separate Dokku apps (Web and Pipeline) from this single repository using the `monorepo` plugin.

```bash
# Link apps to the shared DB
dokku postgres:link psis-db psis-web
dokku postgres:link psis-db psis-pipeline

# Configure subfolder contexts
dokku config:set psis-web APP_CONTEXT=dashboard
dokku config:set psis-pipeline APP_CONTEXT=data-pipeline

```

---

## 📊 Analytics Logic: The Location Quotient

The core metric is the **Location Quotient (LQ)**, calculated in the `fct_location_quotient` dbt model:

* **LQ > 1.2:** Regional Specialization (Green in dashboard)
* **LQ < 0.8:** Under-represented Field (Blue in dashboard)

---

## 🛠 Setup & Installation

### Prerequisites

* Python 3.10+
* PHP 8.x
* PostgreSQL

### Local Development

1. **Clone the repo:** `git clone https://github.com/your-username/psis-specialization.git`
2. **Run Pipeline:** ```bash
cd data-pipeline
pip install -r requirements.txt
python dlt_loader.py
cd dbt_transform && dbt run
```

```


3. **Launch Web:**
Point your local PHP server to the `/dashboard` folder.

---

## 📝 Technical Achievements

* **Data Integrity:** Implemented `dbt tests` (unique, not_null) to ensure no double-counting of StatCan "Total" rows.
* **Performance:** Used dbt **Ephemerals** and **Incremental** models to optimize transformations on large student datasets.
* **DevOps:** Automated deployment via Git hooks and containerized the Python environment to ensure "it works on my machine" works on the server.

---

### Pro-Tip for your Portfolio:

Add a **"Lessons Learned"** section at the bottom. Mentioning that you had to filter out the StatCan "Total" rows to avoid double-counting is a massive "green flag" for hiring managers because it shows you understand the nuances of real-world data.