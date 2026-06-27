Money Laundering Detection
Classify financial transactions as fraudulent (money laundering) or legitimate using transaction metadata, balance changes, and a Random Forest classifier.

This repository is the submission for Project #11 – Money Laundering Detection from the 30 Cybersecurity Projects Using Machine Learning brief.

Table of Contents
Project layout
Dataset
Quick start (macOS / Linux)
Quick start (Windows)
Running the pipeline
Making predictions
Running the unit tests
Notebooks
Results & artefacts
Troubleshooting

Project layout
money-laundering-detection/
├── config/
│ └── config.yaml # paths, hyper-parameters, seeds
├── data/
│ ├── raw/ # place the Kaggle CSV here
│ ├── processed/ # train/val/test splits (generated)
│ ├── sample/ # small bundled sample for quick tests
│ └── README.md # data dictionary
├── models_saved/ # trained model + feature list (generated)
├── notebooks/
│ ├── EDA.ipynb
│ ├── model_training.ipynb
│ └── results_analysis.ipynb
├── results/ # figures & metrics (generated)
├── scripts/
│ ├── run_pipeline.sh # macOS / Linux one-shot runner
│ ├── run_pipeline.bat # Windows equivalent
│ ├── download_data.sh # Kaggle CLI helper (macOS / Linux)
│ └── download_data.bat # Kaggle CLI helper (Windows)
├── src/
│ ├── preprocessing/ # data loading & cleaning
│ ├── feature_engineering/ # transaction feature extraction & ratios
│ ├── models/ # Random Forest factory
│ ├── training/ # end-to-end training driver
│ ├── evaluation/ # metrics + plots
│ ├── utils/ # logger, config loader
│ └── predict.py # command-line predictor
├── tests/ # pytest unit tests (coverage ≥ 60 %)
├── requirements.txt
├── SOURCES.txt # every paper / link / dataset cited
├── LICENSE
└── README.md # you are here

---

## Dataset

We use the Kaggle dataset **“Synthetic Financial Datasets For Fraud Detection”** by NTNU.

> <https://www.kaggle.com/datasets/ealaxi/paysim1>

The file is **`AML_Fraud_full.csv`** — 10 000 rows, 11 columns (transaction type, amounts, balances, and a `isFraud` target: 1 = fraudulent / money laundering, 0 = legitimate).

### Where to put the CSV

Download the CSV and place it in:

data/raw/AML_Fraud_full.csv

If the file is missing, the pipeline automatically falls back to the bundled 200-row sample at `data/sample/sample_aml.csv` so you can still run every command end-to-end.

For detailed licensing and a field-by-field data dictionary, see [`data/README.md`](data/README.md).

---

data/raw/AML_Fraud_full.csv

If the file is missing, the pipeline automatically falls back to the bundled 200-row sample at `data/sample/sample_aml.csv` so you can still run every command end-to-end.

For detailed licensing and a field-by-field data dictionary, see [`data/README.md`](data/README.md).

---

## Quick start (macOS / Linux)

All commands below assume **Python 3.10 or newer**.

```bash
# 1. Clone / unzip the project, then cd into it
cd money-laundering-detection

# 2. Create and activate a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# 3. Install the dependencies
pip install --upgrade pip
pip install -r requirements.txt

# 4. (Optional) place the Kaggle CSV
#    Download 'AML_Fraud_full.csv' from Kaggle and drop it in
#    data/raw/.  If you skip this step the bundled sample is used.

# 5. Train the model (takes ~10-30 seconds on a laptop)
python -m src.training.train

# 6. Make a prediction for a transaction
python -m src.predict --amount 1000000 --type "TRANSFER"

A one-shot wrapper is available at scripts/run_pipeline.sh:
bash scripts/run_pipeline.

Quick start (Windows)
All commands are for PowerShell (works in Windows Terminal, VS Code terminal, or plain PowerShell). Requires Python 3.10+ from python.org or the Microsoft Store.
# 1. Open PowerShell in the project folder
cd money-laundering-detection

# 2. Create and activate a virtual environment
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1

# If PowerShell blocks the activation script, run this once:
#   Set-ExecutionPolicy -Scope CurrentUser RemoteSigned

# 3. Install the dependencies
python -m pip install --upgrade pip
pip install -r requirements.txt

# 4. (Optional) place the Kaggle CSV in data\raw\AML_Fraud_full.csv

# 5. Train the model
python -m src.training.train

# 6. Make a prediction for a transaction
python -m src.predict --amount 1000000 --type "TRANSFER"

A one-shot batch wrapper is available at scripts\run_pipeline.bat
scripts\run_pipeline.bat:
Using Command Prompt (cmd.exe) instead of PowerShell? Replace the activation line with .\.venv\Scripts\activate.bat.

Running the pipeline
The end-to-end pipeline is exposed as a single Python module:
python -m src.training.train
What it does, step by step:

Loads the CSV from data/raw/ (or the bundled sample).
Cleans missing / duplicate rows.
Adds three engineered ratio features (errorBalanceOrig, errorBalanceDest, amountToBalanceRatio).
Stratified 70 / 15 / 15 train / validation / test split.
Fits a Random Forest pipeline (StandardScaler → Random Forest).
Performs 5-fold stratified cross-validation on the training set.
Evaluates on train, validation and test.
Writes artefacts:
+ models_saved/aml_rf_model.joblib — serialised model
+ models_saved/feature_columns.json — canonical feature order
+ data/processed/{train,val,test}.csv — persisted splits
+ results/metrics_summary.json — all metrics in one file
+ results/confusion_matrix.png
+ results/roc_curve.png
+ results/pr_curve.png
+ results/feature_importance.png
+ results/classification_report.txt
(these are meant to be bullet points)

Making predictions
Single transaction:
python -m src.predict --amount 1000000 --type "TRANSFER"

Output:
{
  "amount": 1000000,
  "type": "TRANSFER",
  "prediction": "fraud",
  "probability": 0.97
}

Batch CSV (with the required feature columns):
python -m src.predict --csv data/raw/AML_Fraud_full.csv
 Creates AML_Fraud_full_predictions.csv next to the input file.

Running the unit tests
From the project root:
pytest -v
Or measure coverage:
pytest --cov=src --cov-report=term-missing
Tests cover the feature extractor, preprocessing, and the model pipeline. On a reference run they report > 70 % line coverage.

Results & artefacts
After running python -m src.training.train, the following files are regenerated:
| File | Description |
| --- | --- |
| `models_saved/aml_rf_model.joblib` | trained scikit-learn pipeline |
| `models_saved/feature_columns.json` | canonical feature order |
| `data/processed/train.csv` | stratified training split |
| `data/processed/val.csv` | validation split |
| `data/processed/test.csv` | held-out test split |
| `results/metrics_summary.json` | accuracy / precision / recall / F1 / ROC-AUC on every split |
| `results/confusion_matrix.png` | test-set confusion matrix |
| `results/roc_curve.png` | ROC curve with AUC |
| `results/pr_curve.png` | Precision–Recall curve |
| `results/feature_importance.png` | top-20 feature importances |
| `results/classification_report.txt` | per-class classification report |

Troubleshooting
ModuleNotFoundError: No module named 'src'
: Run commands from the project root, e.g. python -m src.training.train, not python src/training/train.py.

FileNotFoundError: … AML_Fraud_full.csv
: The dataset is not in data/raw/. Either download it from the Kaggle link above or let the pipeline use the bundled sample (it will log a warning and continue).

License
MIT — see LICENSE.
