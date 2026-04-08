# 📦 E-Commerce Data Pipeline (Databricks)

## 🔷 Project Overview

This project implements an end-to-end **Medallion Architecture (Bronze → Silver → Gold)** pipeline for an e-commerce dataset using Databricks. The pipeline ingests raw CSV data, processes it into curated layers, supports streaming ingestion, and orchestrates workflows using Databricks Jobs.

---

## 🏗️ Architecture

```
Raw Data (Volume)
      ↓
Bronze Layer (Raw Delta Tables)
      ↓
Silver Layer (Cleaned & Enriched)
      ↓
Gold Layer (Business Aggregations)
      ↓
Streaming (Incremental Updates)
```

---

## 🧱 Environment Setup

### Catalog & Schemas

```sql
CREATE SCHEMA de_workspace26.michaelBronze;
CREATE SCHEMA de_workspace26.michaelSilver;
CREATE SCHEMA de_workspace26.michaelGold;
```

### Volume (Raw Data Storage)

```sql
CREATE VOLUME de_workspace26.ecommerce_schema_michael.ecommerce_volume_michael;
```

Raw data stored at:

```
/Volumes/de_workspace26/ecommerce_schema_michael/ecommerce_volume_michael/raw/
```

---

## 🟤 Bronze Layer (Raw Ingestion)

### Steps:

* Read CSV files using `spark.read.csv()`
* Add metadata columns:

  * `_ingested_at`
  * `_source_file`
* Write as Delta tables

### Tables:

* `michaelBronze.orders`
* `michaelBronze.customers`
* `michaelBronze.products`

### Key Features:

* Schema inference enabled
* Delta format used
* Overwrite mode for initial load

---

## ⚪ Silver Layer (Transformation)

### Steps:

* Remove duplicates (`dropDuplicates`)
* Convert `order_date` → Date
* Add `revenue = quantity * unit_price`
* Apply window function:

  * `cumulative_revenue` per customer
* Join with customers & products

### Output:

* `michaelSilver.orders` (partitioned by `region`)

---

## 🔄 SCD Type 2 (Customers)

### Implementation:

* Used `MERGE INTO`
* Tracks changes in `loyalty_tier`

### Columns added:

* `is_current`
* `effective_start_date`
* `effective_end_date`

---

## 🟡 Gold Layer (Analytics)

### Views Created:

#### 1. Monthly Revenue by Region

* Aggregated by year, month, region

#### 2. Top Products

* Ranked using `RANK()` by category

---

## 📊 Dashboard (Databricks SQL)

* Bar chart: Monthly revenue by region
* Table: Top 10 products
* Scheduled for daily refresh

---

## 🌊 Streaming Layer (Auto Loader)

### Configuration:

* Format: CSV
* Source: Volume path
* Schema tracking:

  ```
  cloudFiles.schemaLocation
  ```
* Checkpoint:

  ```
  checkpointLocation
  ```

### Key Features:

* Incremental ingestion
* Watermark applied (10 minutes)
* Output mode: append

### Important Fix:

```python
.trigger(availableNow=True)
```

➡️ Ensures stream runs once and stops automatically

---

## 🔁 Checkpoint & Schema Management

| Component          | Purpose                |
| ------------------ | ---------------------- |
| schemaLocation     | Stores schema metadata |
| checkpointLocation | Tracks processed files |

### Reset (for testing only):

```python
dbutils.fs.rm("/tmp/checkpoints/live_orders", True)
```

---

## ⚙️ Workflow Orchestration

### Tasks:

1. Ingest Bronze
2. Transform Silver
3. Build Gold
4. Run Streaming

### Configuration:

* Sequential dependencies
* Job cluster (auto-terminate: 30 mins)
* Retry: 2 (Bronze only)
* Schedule: Daily at 6 AM UTC
* Email alerts on failure

---

## ⚠️ Important Learnings

### 1. Cluster vs Catalog

* Cluster = compute
* Catalog = data namespace
  ➡️ Must always use correct catalog (`de_workspace26`)

---

### 2. Streaming Behavior

* Checkpoint prevents duplicate ingestion
* Deleting checkpoint resets ingestion state

---

### 3. Schema vs Data Paths

* `schemaLocation` → metadata only
* `checkpointLocation` → progress tracking
* Not actual data storage

---

### 4. Production Best Practices

❌ Do NOT:

* Delete tables
* Delete checkpoints randomly
* Use overwrite blindly

✅ Do:

* Use MERGE for incremental updates
* Maintain checkpoints
* Use partitioning and Z-order

---

## 🚀 Final Outcome

* Fully functional **batch + streaming pipeline**
* Clean medallion architecture
* Idempotent ingestion (no duplicates)
* Scalable and production-aligned design

---

## 👨‍💻 Author

Michael
(Data Engineering Pipeline Implementation)
