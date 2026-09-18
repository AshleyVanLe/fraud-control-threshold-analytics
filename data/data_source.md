# Dataset

**IEEE-CIS Fraud Detection**

A transaction fraud dataset released through the IEEE-CIS Fraud Detection competition on Kaggle. The data contains transaction-level activity, fraud labels, payment and product attributes, and identity information for a subset of transactions.

This project uses the labeled training data to examine fraud patterns and simulate how different model risk thresholds trade off fraud capture against legitimate transaction friction.

Raw source files are not redistributed in this repository. This repository contains analysis code, documentation, and final visual outputs only.

---

## Dataset Scope

| **Item** | **Coverage** |
| --- | --- |
| Transactions | 590,540 |
| Fraud transactions | 20,663 |
| Overall fraud rate | 3.50% |
| Relative days | 182 |
| Relative weeks | 26 |
| Transaction grain | One row per transaction |
| Identity-linked transactions | 144,233 |
| Identity coverage | 24.42% |

The dataset does not provide real calendar timestamps. Time-based analysis therefore uses relative days and weeks derived from `TransactionDT`.

---

## Files Used

### `train_transaction.csv`

Transaction-level table containing the fraud target and transaction, product, card, time, and anonymized variables.

**Grain:** one row per transaction  
**Primary key:** `TransactionID`  
**Target:** `isFraud`

### `train_identity.csv`

Identity table containing additional identity and device information for a subset of transactions.

**Grain:** one row per identity-linked transaction  
**Join key:** `TransactionID`

The two tables are joined using:

```text
train_transaction.TransactionID
                 ↕
train_identity.TransactionID
```

Identity information is not available for every transaction.

The competition test files were not used because this project analyzes observed fraud outcomes and the test set does not provide the `isFraud` target.

---

## Fields Used in the Analysis

The project intentionally uses a subset of the available variables.

### Transaction and Target

- `TransactionID`
- `TransactionDT`
- `TransactionAmt`
- `isFraud`

### Product and Card

- `ProductCD`
- `card4`
- `card6`

### Identity and Device

Selected identity and device variables were reviewed where coverage allowed.

Because identity information is available for only a subset of transactions, identity-based comparisons are not treated as representative of the full transaction population.

### Derived Fields

Several fields were created during the analysis:

- `RelativeDay`
- `RelativeWeek`
- `FraudStatus`
- `FraudValue`
- transaction amount bands
- model risk score

The model risk score is used to rank transactions and simulate fraud-control thresholds. It is treated as a **risk-ranking signal**, not as a perfectly calibrated probability of fraud.

---

## Data Validation

Before analyzing fraud patterns or fitting a risk model, I checked the analytical grain and internal consistency of the source files.

### Transaction Table

- 590,540 transaction records
- 590,540 unique `TransactionID` values
- no duplicate transaction IDs
- 20,663 transactions labeled as fraud
- overall fraud rate of approximately 3.50%

### Identity Table

- 144,233 identity records
- identity information available for approximately 24.42% of transactions

Not every transaction has a corresponding identity record.

Identity coverage also differs across parts of the transaction data, so identity-linked findings are interpreted within the observed subset rather than automatically generalized to all transactions.

### Missing Data

Missingness is substantial across several variables in the IEEE-CIS dataset.

Rows were not automatically removed simply because one or more fields were missing. Doing so would substantially reduce the analytical population and could introduce additional selection bias.

Missing values were instead handled according to the variables and modeling steps in which they were used.

### Transaction Amount

`TransactionAmt` is strongly right-skewed.

The observed median is approximately **68.77**, compared with a mean of approximately **135.03**.

Large transaction values were not automatically treated as errors or removed as outliers. Transaction count and transaction value are analyzed separately where appropriate.

---

## Time Construction

`TransactionDT` does not represent an actual calendar timestamp. It is a time delta from an undisclosed reference point.

For this project, it was converted into relative periods:

```text
TransactionDT
      │
      ├── RelativeDay
      │
      └── RelativeWeek
```

The resulting transaction history covers **182 relative days** and **26 relative weeks**.

No attempt is made to assign actual dates, months, seasons, holidays, or external events to these periods.

This allows changes in fraud exposure to be examined over the observed sequence without introducing unsupported calendar assumptions.

---

## Fields Used With Caution

### `ProductCD`

The dataset provides five anonymized product codes:

- C
- H
- R
- S
- W

Their underlying business meanings are not established in this project.

The analysis can compare observed behavior across these categories, but it does not assign unsupported product names or interpretations to them.

### Identity Variables

Identity information is available for approximately **24%** of transactions and coverage is not uniform across the dataset.

Device and identity patterns are therefore interpreted as characteristics of the observed identity subset rather than the full transaction population.

### `TransactionDT`

`TransactionDT` is used only to construct relative time periods.

Real calendar dates are not inferred.

### `TransactionAmt`

The project does not assign a currency to `TransactionAmt` because the available dataset information used in this analysis does not establish one.

Transaction-value measures are therefore presented without a currency symbol.

### Model Risk Score

The model output is used as a risk-ranking signal for threshold analysis.

Because the modeling approach addresses a highly imbalanced fraud target and probability calibration is not established as a primary objective, the score is not presented as a literal probability that a transaction is fraudulent.

---

## Simulated Customer Friction

The source data provides observed fraud labels but does **not** provide actual fraud-control decisions.

There is no field indicating whether a transaction was:

- approved
- declined
- blocked
- sent to manual review
- challenged with additional authentication

The dataset also does not contain customer complaints or another direct measure of customer friction.

For this reason, **legitimate transaction friction is simulated**.

At a hypothetical risk threshold:

```text
model risk score ≥ threshold
              │
              ▼
      transaction flagged
              │
       ┌──────┴──────┐
       ▼             ▼
    Fraud        Legitimate
    caught       interrupted
```

A legitimate transaction with a model risk score at or above the selected threshold is counted as **legitimate interrupted**.

This represents the legitimate activity that would be affected under the simulated control policy.

It does not mean the transaction was actually declined or that the customer experienced observed friction.

---

## Analytical Boundaries

This project supports statements about fraud patterns and simulated threshold behavior **within this dataset and modeling setup**.

It does not establish:

- that any threshold is optimal for a real financial institution
- that a flagged legitimate transaction would actually be declined
- that a flagged transaction created observed customer friction
- the operational cost of reviewing a flagged transaction
- the financial loss associated with every fraud transaction
- the business cost of interrupting legitimate activity
- the real calendar dates represented by `TransactionDT`
- the currency represented by `TransactionAmt`
- the business meaning of the anonymized product codes
- causal relationships between transaction characteristics and fraud
- that the results generalize to another transaction population

These boundaries are maintained throughout the notebooks and dashboard.

The threshold analysis should therefore be interpreted as a **fraud-control decision simulation**, not as an evaluation of an existing bank fraud-control system.

---

## Provenance

**Dataset:** IEEE-CIS Fraud Detection  
**Competition host:** Kaggle  
**Competition organization:** IEEE Computational Intelligence Society  
**Original data provider:** Vesta Corporation  
**Downloaded:** September 2026  
**Usage terms:** Subject to Competition Rules

**Dataset page:**  
https://www.kaggle.com/competitions/ieee-fraud-detection/data

**Competition rules:**  
https://www.kaggle.com/competitions/ieee-fraud-detection/rules

The raw competition data and derived transaction-level files are not redistributed in this repository.

The source data remains subject to the terms established by its original publisher and Kaggle and is not covered by the MIT License applied to this repository's original code.

---

## Reproducing the Project

To reproduce the analysis:

1. Register for Kaggle and accept the IEEE-CIS Fraud Detection competition rules.
2. Download the competition data directly from Kaggle.
3. Store `train_transaction.csv` and `train_identity.csv` locally.
4. Update the local data path at the beginning of the notebooks if necessary.
5. Run the notebooks in numerical order.

Expected local structure:

```text
data/
├── raw/
│   ├── train_transaction.csv
│   └── train_identity.csv
│
└── data_source.md
```

The `data/raw/` directory and derived row-level datasets are excluded from version control.

This keeps the repository focused on reproducible analytical work while preserving the original dataset's distribution terms.
