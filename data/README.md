# Data dictionary
This project uses the Kaggle dataset

Synthetic Financial Datasets For Fraud Detectionby NTNUhttps://www.kaggle.com/datasets/ealaxi/paysim1

License: CC BY 4.0. You must credit the original authors when redistributing the dataset or derived artefacts.

# Files
File	Purpose
raw/AML_Fraud_full.csv	Full dataset. Place it here after downloading from Kaggle.
sample/sample_aml.csv	200-row stratified sample bundled with the repo.
processed/train.csv	Training split produced by src/training/train.py.
processed/val.csv	Validation split.
processed/test.csv	Test split.

# Schema
Column	Type	Description
step	int	Maps a unit of time in the real world. 1 step = 1 hour.
type	string	Type of transaction (CASH-IN, CASH-OUT, DEBIT, PAYMENT, TRANSFER).
amount	float	Amount of the transaction in local currency.
nameOrig	string	Customer who started the transaction.
oldbalanceOrg	float	Initial balance before the transaction.
newbalanceOrig	float	New balance after the transaction.
nameDest	string	Customer who is the recipient of the transaction.
oldbalanceDest	float	Initial recipient balance before the transaction.
newbalanceDest	float	New recipient balance after the transaction.
isFraud	0/1	Target. 0 = legitimate, 1 = fraudulent / money laundering.
isFlaggedFraud	0/1	Legal compliance flag (attempted illegal transfer > 200,000).
Engineered features added by this project
Column	Formula
errorBalanceOrig	newbalanceOrig + amount - oldbalanceOrg
errorBalanceDest	oldbalanceDest + amount - newbalanceDest
amountToBalanceRatio	amount / (oldbalanceOrg + 1)
These capture balance reconciliation errors and transaction proportionality, which are highly indicative of structuring and layering in money laundering.

# Class distribution
The dataset is highly imbalanced, typical of financial fraud:

~98% legitimate transactions
~2% fraudulent / money laundering transactions
Because of this, metrics like Precision, Recall, F1-score, and ROC-AUC are prioritized over pure Accuracy.
