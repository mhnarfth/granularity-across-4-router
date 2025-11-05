
# 📈 REN Traffic Forecasting: A Granularity & Architecture Study

This repository contains the complete workflow for the journal paper extending our work on REN traffic forecasting. The core of this research is a large-scale, systematic study investigating two primary questions:

1.  **The Impact of Granularity:** How does the time-series aggregation level (from 5-minute intervals to daily summaries) affect the prediction accuracy of deep learning models on REN traffic?
2.  **Architectural Validation:** Is our proposed hybrid GRU-LSTM architecture empirically superior to its standalone components (GRU-only, LSTM-only) and a strong, modern deep learning baseline?

The project is structured as a sequence of four Jupyter Notebooks, designed for clarity, reproducibility, and modularity.

---

## 📂 Project Structure

The project is organized into a clean workflow. Notebooks are numbered to guide the execution order. All outputs are generated into a structured `outputs/` directory.

```
journal_extension_project/
│
├──  notebooks/
│   ├── 01_ETL.ipynb
│   ├── 02_EDA.ipynb
│   ├── 03_Experiments.ipynb
│   └── 04_Results_and_Visualization.ipynb
│
└── outputs/
    ├── eda/                      # Exploratory plots
    │   └── *.png, *.pdf
    │
    ├── predictions/              # Detailed y_true vs. y_pred data
    │   └── *.parquet
    │
    ├── loss_curves/              # (Optional) Diagnostic training loss plots
    │   └── *.png
    │
    └── results/
        ├── all_granularity_study_results.csv   # Master CSV with all metrics
        │
        └── figures/                  # Final, paper-quality figures
            └── *.png, *.pdf
```

---

## 📊 Data Provenance and Access

The data used in this study is proprietary network flow data from the Internet2 backbone and is subject to strict access controls.

**Important:** The raw and processed datasets are stored on the university's high-performance storage system (NRDstor) and are **not accessible to outsiders or the general public**. The paths are hardcoded in the notebooks for internal use.

*   **Raw Data Source:**
    *   The original, un-aggregated NetFlow data for each router resides on NRDstor.
    *   **Location:** `/mnt/nrdstor/ramamurthy/mhnarfth/internet2/parquet`

*   **Processed "Golden Source" Data:**
    *   After being processed by `01_ETL.ipynb`, the clean, consolidated dataset for this study is also stored on NRDstor to manage storage and facilitate access from compute nodes.
    *   **Location:** `/mnt/nrdstor/ramamurthy/mhnarfth/journal_extension_data/journal_study_4r_57d_unaggregated.parquet`

---

## 🚀 Execution Workflow

This project is designed to be run sequentially. Please execute the notebooks in numerical order.

### **Step 1: Data Preparation (`01_ETL.ipynb`)**

> **Objective:** To create a single, clean "golden source" dataset from the raw network flows.

*   **What it Does:** This notebook connects to the raw Internet2 data on NRDstor. It processes the data for four specifically chosen routers, cleans it, and saves a single, consolidated, un-aggregated Parquet file back to NRDstor. This is the foundational dataset for the entire study.
*   **Routers Processed:**
    *   `dallas` (High-Traffic Hub)
    *   `atlanta` (High-Traffic Hub)
    *   `elpaso` (Lower-Traffic Regional)
    *   `boston` (Lower-Traffic Regional)
*   **Input:** Raw Parquet files from the NRDstor path mentioned in the "Data Provenance" section.
*   **Output:** A single, clean Parquet file saved to its own NRDstor path, also mentioned above.
*   **▶️ How to Run:** Open `notebooks/01_ETL.ipynb`, verify the path knobs, and run all cells. **This only needs to be run once.**

### **Step 2: Exploratory Data Analysis (`02_EDA.ipynb`)**

> **Objective:** To visualize the characteristics of our chosen routers and justify their selection.

*   **What it Does:** This notebook loads the "golden source" file and generates high-level plots comparing the traffic volumes and daily patterns of the four routers. It provides the initial insights that motivate our study.
*   **Input:** The "golden source" Parquet file from Step 1.
*   **Output:** A set of exploratory plots saved to `outputs/eda/`.
*   **▶️ How to Run:** Open `notebooks/02_EDA.ipynb` and run all cells. This notebook can be re-run anytime to adjust plot aesthetics.

### **Step 3: Model Experiments (`03_Experiments.ipynb`)**

> **Objective:** To conduct the full suite of computational experiments, training hundreds of models to explore our research questions. **This is the main computational step.**

*   **What it Does:** This is the workhorse of the project. It systematically iterates through every combination of router, granularity, and model. It leverages the available **GPU** for accelerated training. For each run, it performs a robust 70/15/15 chronological train-validation-test split and saves all performance metrics and predictions.
*   **Experimental Space:**
    *   **Models Benchmarked:**
        1.  **`hybrid_gru_lstm`**: Our proposed hybrid model.
        2.  **`gru_only`**: Standalone GRU model (for ablation).
        3.  **`lstm_only`**: Standalone LSTM model (for ablation).
        4.  **`nbeats`**: A strong, modern deep learning baseline (non-RNN).
    *   **Granularities Tested:** `['5min', '15min', '30min', '45min', '1H', '2H', '4H', '6H', '12H', '1D']`
*   **Input:** The "golden source" Parquet file from Step 1.
*   **Outputs:**
    1.  `outputs/results/all_granularity_study_results.csv`: A master CSV file with all performance metrics (MAE, RMSE, sMAPE, R², etc.).
    2.  `outputs/predictions/`: A directory filled with Parquet files, one for each experiment, containing detailed `y_true` and `y_pred` data for plotting.
*   **▶️ How to Run:** Open `notebooks/03_Experiments.ipynb`. Review the configuration knobs at the top to set the scale of the run (for a full run, use all settings; for a quick test, reduce the lists). Execute all cells and monitor the progress. **This will take a significant amount of time.**

### **Step 4: Results & Visualization (`04_Results_and_Visualization.ipynb`)**

> **Objective:** To synthesize the vast amount of data from our experiments into a small set of clear, insightful, and publication-quality figures and tables.

*   **What it Does:** This is the final analysis stage. It loads the master metrics CSV and specific prediction files to create the core visuals that tell our scientific story. It answers our research questions by plotting performance against granularity, comparing models, and visualizing prediction accuracy.
*   **Inputs:** All files generated by `03_Experiments.ipynb`.
*   **Outputs:** The final figures for the paper (e.g., "Granularity vs. Performance," "Model Ablation Study") saved to `outputs/results/figures/`, and summary tables printed in the notebook.
*   **▶️ How to Run:** Open `notebooks/04_Results_and_Visualization.ipynb`. Run the cells sequentially. Each plotting block has its own aesthetic knobs that can be tweaked to perfect the final figures for the manuscript.

---
