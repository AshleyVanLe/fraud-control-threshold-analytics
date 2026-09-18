# Data Source

This project uses the IEEE-CIS Fraud Detection dataset from the Kaggle competition **IEEE-CIS Fraud Detection**.

Dataset:
IEEE-CIS Fraud Detection

Source:
Kaggle Competition Data

Files used:
- `train_transaction.csv`
- `train_identity.csv`

The transaction dataset contains 590,540 transactions and includes the binary fraud label `isFraud`. Identity information is available for a subset of transactions.

## Data Access

The raw competition files are not included in this repository.

To reproduce the analysis, download the dataset directly from the IEEE-CIS Fraud Detection competition page on Kaggle and place the required files in your local data directory.

## Important Data Limitations

The dataset does not provide:

- actual bank approve, decline, or manual-review decisions
- customer complaints or directly observed customer friction
- persistent clean customer identifiers
- real calendar timestamps
- documented meanings for all anonymized product categories

For this reason, **customer friction in this project is simulated**, not directly observed. It represents legitimate transactions that would be interrupted under a hypothetical fraud-control threshold.

`TransactionDT` is also converted into relative days and weeks rather than interpreted as a real calendar date.

## Repository Policy

Raw competition data and derived transaction-level files are intentionally excluded from this public repository. This repository contains the analysis code, methodology, and final visual outputs.
