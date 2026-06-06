# Volatility Surface Modelling: The Trinity Ensemble

This repository contains a pipeline designed to predict missing Implied Volatility (IV) values across sparse option chains. The methodology minimizes Mean Squared Error (MSE) utilizing a "Trinity Ensemble" of non-linear mathematical interpolators.

## 🏗️ Pipeline Architecture

To ensure strict reproducibility and prevent data leakage (look-ahead bias) within a notebook environment, the system is modularized into three sequential Jupyter Notebooks:

1. **`01_setup.ipynb` (Data Ingestion & Partitioning):** Parses the raw Kaggle matrix, engineers Black-Scholes coordinates (Log-Moneyness), and strictly isolates known market quotes from the testing targets.
2. **`engine.ipynb` (The Mathematical Core):** Evaluates the volatility surface cross-sectionally. It deploys three geometric interpolators and blends them to balance macro-market stability with micro-structural precision.
3. **`submission.ipynb` (Mapping & Export):** Bridges the calculated features back to the required target submission schema safely.

## 🧮 The "Trinity" Methodology

Options pricing datasets are heavily penalized by outlier predictions (jagged wings). Standard Machine Learning algorithms (like XGBoost or Random Forests) often fail to respect the physical convexity of the volatility smile. 

This model solves the "Outlier Trap" by ensembling three geometric bases:
* **Natural Cubic Splines (25% Weight):** Anchors the macro-trend and overall smoothness of the options chain.
* **PCHIP Interpolation (35% Weight):** Applies shape-preserving bounds that prevent the splines from "wiggling" or overshooting on sparse data intervals.
* **Akima 1D Splines (40% Weight):** Acts as a local "sniper," perfectly threading the interior points and cleanly ignoring bid-ask spread anomalies without disrupting the global curve.

## 📂 Repository Structure

```text
├── dataset.csv               # The raw matrix dataset (Not included in repo)
├── 01_setup.ipynb            # Pre-processing & environment setup
├── engine.ipynb              # Core predictive mathematical logic
├── submission.ipynb          # Output formatter & Kaggle mapping
└── README.md                 # Project documentation
```
## ⚙️ Installation & Requirements

Ensure you have Python 3.8+ installed. The pipeline requires a Jupyter environment and the following standard scientific computing libraries:

```bash
pip install pandas numpy scipy jupyterlab
```
## 🚀 Usage Guide

**Step 1: Add your data**
Place the raw `dataset.csv` file into the root directory of the project.

**Step 2: Execute Notebooks Sequentially**
Because Jupyter Notebooks retain memory states, you **must** run these files in strict order. Open each notebook and select **Kernel -> Restart & Run All**.

1. Run `01_setup.ipynb` — *Generates train_dataset.csv and test_dataset.csv*
2. Run `engine.ipynb` — *Calculates the surfaces and generates filled_dataset.csv*
3. Run `submission.ipynb` — *Generates the final submission.csv*

*Note: Before committing any changes to GitHub, please ensure you clear your notebook outputs (`Kernel -> Restart & Clear Output`) to prevent pushing large data previews to the repository.*

## 🛡️ Leakage Prevention Guarantee
By isolating the processing engine from the dataset compilation across different notebooks, the `engine.ipynb` environment is mathematically forced to rely only on the known coordinates provided by the training split. It cannot reference future or adjacent target columns, ensuring that local Cross-Validation perfectly mimics the live, out-of-sample testing environment.
