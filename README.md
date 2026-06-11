# Large-Scale Soil Analytics Engine

A distributed data processing and machine learning pipeline built on **Apache Spark**, designed to handle billion-row scale agricultural sensor datasets. Built for the **Technex 2025 — 1 Billion Row Challenge (Final Round)** at Indian Institute of Technology BHU.

---

## Overview

This pipeline ingests massive CSV datasets containing soil sensor readings and runs a full analytics workflow end-to-end: schema-validated ingestion, anomaly filtering, multi-column sorting, descriptive statistics, missing value interpolation, linear regression modeling, and visualization — all within a single Spark session with optional GPU acceleration via NVIDIA RAPIDS.

---

## Pipeline Stages

### Task 1 — Data Ingestion & Preparation
- Loads a large CSV file with an explicit schema (no type inference overhead)
- Applies range-based filters on all 7 sensor columns at read time
- Uses stratified sampling to estimate total row count without a full scan

### Task 2 — Anomaly Filtering
- Computes `mean` and `stddev` of `soil_moisture` on a 0.1% sample
- Drops rows outside **±3 standard deviations** from the mean

### Task 3 — Multi-Column Sorting & Filtering
- Sorts a sample by `timestamp`, `soil_moisture`, and `temperature`
- Applies compound filter: `soil_moisture > 80` AND `pH < 5`

### Task 4 — Statistical Analysis
Computes the following for both `carbon_percent` and `nitrogen_percent`:
- Mean, Median, Standard Deviation, Q1 (25th percentile), Q3 (75th percentile)

### Task 5 — Interpolation & Prediction
- Imputes null `carbon_percent` values as `soil_moisture * 0.05`
- Builds a **Linear Regression** model (Spark ML) predicting `carbon_percent` from `soil_moisture` and `temperature`
- Trains on 80% / evaluates on 20% split of a 0.1% sample

### Task 6 — Visualization
- Generates a bar chart of nutrient statistics (`carbon_percent` vs `nitrogen_percent`) in a background thread
- Saves output as `stats_plot.png`

---

## Dataset Schema

| Column | Type | Valid Range |
|---|---|---|
| `timestamp` | Integer | — |
| `soil_moisture` | Float | 0 – 100 |
| `soil_water_content` | Float | 0 – 100 |
| `carbon_percent` | Float | 0 – 10 |
| `nitrogen_percent` | Float | 0 – 5 |
| `atmospheric_humidity` | Float | 0 – 100 |
| `temperature` | Float | 0 – 40 |
| `pH` | Float | 4 – 9 |

> The raw `combined.csv` file is not included in this repository due to its size.

---

## GPU Acceleration

The pipeline auto-detects CUDA availability via `torch.cuda.is_available()` or `nvidia-smi`. If a compatible GPU is found, it applies the [NVIDIA RAPIDS Accelerator for Apache Spark](https://nvidia.github.io/spark-rapids/) plugin.

Required for GPU mode:
- RAPIDS JAR: `rapids-4-spark_2.12-24.04.1.jar`
- Compatible NVIDIA GPU with CUDA drivers installed
- RAPIDS-enabled Spark cluster configuration

CPU-only mode works out of the box without any additional setup.

---

## Requirements

```
pyspark==3.5.1
pandas
numpy
matplotlib
streamlit
ipykernel
```

Install with:

```bash
pip install -r requirement.txt
```

---

## Spark Configuration

| Parameter | Value |
|---|---|
| Master | `local[6]` |
| Driver Memory | 5 GB |
| Executor Memory | 5 GB |
| Shuffle Partitions | 20 |
| Default Parallelism | 12 |
| Adaptive Query Execution | Enabled |

---

## Running

1. Clone the repository:
   ```bash
   git clone https://github.com/aadesh006/Large-Scale-Soil-Analytics-Engine.git
   cd Large-Scale-Soil-Analytics-Engine
   ```

2. Install dependencies:
   ```bash
   pip install -r requirement.txt
   ```

3. Place your dataset at the path referenced in the notebook (`combined.csv`) or update the `file_path` variable accordingly.

4. (GPU only) Update `rapids_jar_path` and `temp_dir_path` in the config block to match your local paths.

5. Open and run `main.ipynb` in Jupyter or VS Code.

---

## Output

- Console logs for each pipeline stage with row samples
- Descriptive statistics for `carbon_percent` and `nitrogen_percent`
- Linear regression predictions on the test split
- `stats_plot.png` — bar chart of nutrient statistics
- Total runtime reported at the end

---

## Context

This project was submitted for **Round 2 of the 1 Billion Row Challenge** at Technex 2025, IIT (BHU) Varanasi's annual techno-management fest. The challenge required processing and deriving insights from a dataset on the scale of 1 billion rows under time and resource constraints.

---

## Tech Stack

`Apache Spark 3.5` · `PySpark ML` · `NVIDIA RAPIDS` · `pandas` · `NumPy` · `Matplotlib` · `Python 3`
