# YouTube API — ELT Pipeline 🎬

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)
![Airflow](https://img.shields.io/badge/Airflow-Orchestration-017CEE?logo=apacheairflow)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-336791?logo=postgresql)
![GitHub Actions](https://img.shields.io/badge/CI--CD-GitHub%20Actions-2088FF?logo=githubactions)

---

## 📌 Motivation

The aim of this project is to get hands-on experience with industry-standard **data engineering tools** such as Python, Docker, and Airflow to build a production-grade **ELT (Extract → Load → Transform) data pipeline**.

To make the pipeline more robust, best practices of **unit testing**, **data quality testing**, and **CI/CD (Continuous Integration / Continuous Deployment)** are also implemented.

---

## 📊 Dataset

The data source for this project is the **YouTube Data API v3**.

Data is pulled from the popular YouTube channel **MrBeast**.

> ℹ️ This project can be replicated for **any YouTube channel** — simply update the `CHANNEL_HANDLE` variable in the config.

### Variables Extracted

| # | Field | Description |
|---|-------|-------------|
| 1 | `video_id` | Unique YouTube video identifier |
| 2 | `title` | Title of the video |
| 3 | `publishedAt` | Upload date of the video |
| 4 | `duration` | Duration of the video |
| 5 | `viewCount` | Total views |
| 6 | `likeCount` | Total likes |
| 7 | `commentCount` | Total comments |

---

## 🏗️ Architecture & Summary

```
YouTube API
     │
     ▼
[Extract] Python Scripts
     │
     ▼
[Load] Staging Schema (PostgreSQL)
     │
     ▼
[Transform] Python Transformations
     │
     ▼
[Load] Core Schema (PostgreSQL)
     │
     ▼
[Validate] Data Quality Tests (SODA)
     │
     ▼
Ready for Analysis ✅
```

### Pipeline Steps

1. **Extract** — YouTube API is called using Python scripts to fetch raw video data
2. **Load (Staging)** — Raw data is loaded into the `staging schema` of a dockerized PostgreSQL database
3. **Transform & Load (Core)** — Minor transformations are applied via Python scripts and data is loaded into the `core schema`

> The **first run** performs a full initial data load. All **subsequent runs** perform an **upsert** — updating existing records and inserting new ones.

---

## 🛠️ Tools & Technologies

| Category | Tool |
|----------|------|
| Containerization | Docker, Docker Compose |
| Orchestration | Apache Airflow |
| Data Storage | PostgreSQL |
| Languages | Python, SQL |
| Testing | pytest, SODA Core |
| CI/CD | GitHub Actions |

---

## 🐳 Containerization

Airflow is deployed on Docker using the [official docker-compose.yaml](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html) with the following customizations:

- A custom **extended Docker image** is built using a `Dockerfile` and pushed to **Docker Hub** via the GitHub Actions CI/CD workflow
- **Database connections** are passed as environment variables in URI format:
  ```
  AIRFLOW_CONN_{CONN_ID}
  ```
- **Airflow Variables** are specified as:
  ```
  AIRFLOW_VAR_{VARIABLE_NAME}
  ```
- A **Fernet Key** is used to encrypt sensitive credentials in connection and variable configs

---

## 🔁 Orchestration (Airflow DAGs)

Three DAGs run sequentially. Access the Airflow UI at: **http://localhost:8080**

| DAG | Purpose |
|-----|---------|
| `produce_json` | Calls YouTube API and produces a raw JSON file |
| `update_db` | Processes JSON and inserts data into staging + core schemas |
| `data_quality` | Runs data quality checks on both database layers |

---

## 🗄️ Data Storage

Data is stored in a **dockerized PostgreSQL** database with two schemas:

- `staging` — raw, unprocessed data
- `core` — cleaned and transformed data, ready for analysis

To query the data, you can either:
- Access the PostgreSQL container directly using `psql`
- Use a database management tool like **DBeaver** to run SQL queries

---

## ✅ Testing

| Type | Tool | Purpose |
|------|------|---------|
| Unit Testing | `pytest` | Tests individual Python functions and logic |
| Data Quality Testing | `SODA Core` | Validates data integrity in staging and core schemas |

---

## ⚙️ CI/CD

CI/CD is implemented using **GitHub Actions**.

It automates the following whenever code changes are pushed:

1. Builds and pushes the custom Docker image to Docker Hub
2. Runs the Docker containers via `docker-compose`
3. Validates that all Airflow DAGs are working as expected

> This ensures that any changes to Airflow code, Docker configs, or Python packages don't break the pipeline.

---

## 🚀 Getting Started

### Prerequisites

- Docker & Docker Compose installed
- YouTube Data API v3 key
- Python 3.10+

### Setup

1. Clone the repository
   ```bash
   git clone https://github.com/<your-username>/<your-repo>.git
   cd <your-repo>
   ```

2. Create a `.env` file in the root directory
   ```
   API_KEY=your_youtube_api_key_here
   ```

3. Start all containers
   ```bash
   docker-compose up -d
   ```

4. Open Airflow UI at `http://localhost:8080` and trigger the DAGs in order:
   - `produce_json` → `update_db` → `data_quality`

---

## 📁 Project Structure

```
.
├── dags/
│   ├── produce_json.py
│   ├── update_db.py
│   └── data_quality.py
├── scripts/
│   ├── extract.py
│   ├── load_staging.py
│   └── transform_load_core.py
├── tests/
│   └── test_pipeline.py
├── .github/
│   └── workflows/
│       └── ci_cd.yaml
├── Dockerfile
├── docker-compose.yaml
├── .env
└── README.md
```

---

## 📄 License

This project is intended for **educational use only**.  
Redistribution, resale, or public sharing is not permitted.

The `docker-compose.yaml` is derived from the [Apache Airflow](https://airflow.apache.org/) project and is licensed under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).
