### # Project README

## CN7030 – Machine Learning on Big Data (Group 4)
### DDoS Network Flow Detection & Duration Prediction (Leakage-Free)

This project presents a robust, leakage-free PySpark and scikit-learn machine learning pipeline for classifying DDoS attacks and predicting network flow durations using the balanced DDoS Network Flow Dataset (`final_dataset.parquet`).

---

## 📂 Repository & Output Structure
All pipeline outputs are automatically saved to `CN7030_Group4_outputs/no_leakage/` to preserve a strict separation from legacy runs:
- `tables/`: Cleaned and formatted dataframes saved as CSV.
- `figures/`: High-resolution figures (PNG) for reporting.
- `reports/`: Complete run history logs (`classification_runs.csv`, `regression_runs.csv`) and a unified Microsoft Word report (`CN7030_Group4_model_runs.docx`).
- `models/`: Serialized Spark PipelineModels (`final_classifier_*` and `final_regressor_*`).
- `splits/`: Group-aware and time-based split partitions for robust evaluation.

---

## 🛠️ Pipeline Tasks Summary

### 1. Data Cleaning & Schema Standardization (Tasks 1–4)
- **Schema Definition**: Explicitly mapped data types (e.g., ports/protocols as `String`, counters as `Long`, rates as `Double`, TCP flags as `Integer`).
- **Timestamp Parsing**: Handled mixed AM/PM and 24-hour timestamp formats dynamically.
- **Zero-Padding & Duplicates**: Sanitized IP addresses and dropped exact duplicate records.
- **Hidden Missing Values**: Replaced physically impossible inputs (e.g., negative inter-arrival times) with nulls, and capped rates resolving division-by-zero infinities.

### 2. Leakage Mitigation (Task 6)
We explicitly dropped columns containing capture/testbed fingerprints to prevent the models from memorizing artifacts rather than behavioral patterns:
- `Fwd Seg Size Min`, `Init Fwd Win Byts`, `Init Bwd Win Byts`, `init_fwd_win_missing`, `init_bwd_win_missing`, `src_port_range`, and `rate_missing` are removed from all modeling tables.

### 3. Model Training & Strict Split Strategy (Task 10 & 12)
To ensure generalizability, models are trained and tested on two separate split schemes:
- **Group-Random Split (70/15/15)**: Groups related flows from the same connection together using a hashed `flow_group` key to avoid connection-level memorization.
- **Time-Based Split**: Sorts flows by timestamp within classes to test performance on future captures (cross-capture generalization).

### 4. Machine Learning Algorithms (Tasks 17–29)
- **Classification**: Majority baseline, Logistic Regression (with class-weighting and cut-off analysis), Decision Trees, Random Forests, Gradient-Boosted Trees, and Multi-Layer Perceptrons (MLP).
- **Regression**: Predicts log-transformed `Flow Duration` (capped to prevent mathematical extrapolation overflow) using Linear Regression, Random Forest Regressor, Gradient-Boosted Regressor, and scikit-learn's MLPRegressor.

### 5. Live Streaming & Visual Dashboard (Task 31)
- Uses Spark Structured Streaming to monitor a folder (`stream_in`) containing simulated flow batches.
- Employs a prediction-only pipeline configuration to classify live incoming flows.
- Generates a live Matplotlib dashboard tracking per-batch accuracy, running confusion matrices, prediction volumes, and probability confidence intervals.
