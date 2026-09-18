# Fraud Signals vs Customer Friction

### Fraud risk modeling and threshold analytics for transaction control decisions

> **Can a fraud control adapt to changing transaction risk without creating unnecessary friction for legitimate activity?**

Fraud models are often evaluated by how well they detect fraud. But once a risk score becomes a control decision, another problem appears: catching more fraud can also interrupt more legitimate transactions.

This project uses the **IEEE-CIS Fraud Detection** dataset to examine that trade-off.

I first explored how observed fraud exposure changes across transaction activity, relative time, anonymized product groups, and payment characteristics. I then built a baseline fraud risk model and used its scores to simulate how different control thresholds change fraud capture, false positives, precision, and transaction value affected.

The goal is not to find a universal "best" threshold. It is to show why fraud control is a decision problem, not only a classification problem.

---

## The Question

The project started with two questions:

**How stable is fraud risk across the transaction environment?**

and then:

**Does one global risk threshold create the same trade-off between fraud capture and legitimate transaction friction?**

This led to a two-part analysis:

1. **Risk monitoring:** identify where observed fraud pressure changes.
2. **Control simulation:** examine what happens when a model risk score is converted into a transaction-control threshold.

---

## Dataset

The analysis uses the training data from the **IEEE-CIS Fraud Detection** competition.

| | |
|---|---:|
| Transactions | 590,540 |
| Fraud transactions | 20,663 |
| Observed fraud rate | 3.50% |
| Relative days | 182 |
| Relative weeks | 26 |
| Identity-linked transactions | 144,233 |
| Identity coverage | 24.42% |

Two source files are used:

- `train_transaction.csv`
- `train_identity.csv`

`TransactionID` provides the transaction-level key and `isFraud` provides the observed binary fraud label.

The raw competition data is **not included in this repository**. See [`data/data_source.md`](data/data_source.md) for provenance, field usage, validation, and analytical limitations.

---

## 1. Mapping Fraud Pressure

Before modeling risk, I first asked whether a portfolio-level fraud rate was enough to describe the transaction environment.

It was not.

The overall observed fraud rate is **3.50%**, but fraud exposure changes across relative time, anonymized product groups, card characteristics, and transaction size.

Weekly fraud rates range from roughly **2.1% to 5.1%**, showing that fraud pressure is not constant over the observed transaction sequence.

Product differences are also substantial. For example, the anonymized `ProductCD = C` group has an observed fraud rate of about **11.69%**, while `ProductCD = W` is about **2.04%**. At the same time, W represents most of the transaction population.

Fraud frequency and transaction value also tell slightly different stories:

- **Fraud transaction rate:** 3.50%
- **Fraud value rate:** 3.87%

This matters because reducing the number of fraudulent transactions and reducing transaction value exposed to fraud are not necessarily the same objective.

### Transaction Risk Monitor

<img width="1370" height="770" alt="01 Risk Monitor" src="https://github.com/user-attachments/assets/cdda783a-3c64-4b0f-a729-828caa716940" />

The first dashboard acts as the descriptive monitoring layer of the project.

It combines:

- weekly transaction volume and fraud pressure
- fraud value exposure
- product-level fraud rates
- product × week variation
- card type and card network profiles
- transaction-size risk
- transaction-level exploration

The main takeaway is not that one transaction group is inherently risky. It is that **a single portfolio average hides meaningful variation in the transaction environment.**

---

## 2. Building a Risk Ranking

The next step was to test whether transaction signals could rank fraud risk well enough to support a threshold analysis.

I used an ordered temporal split rather than randomly mixing transactions across time:

| Split | Relative Weeks |
|---|---|
| Train | 1–18 |
| Validation | 19–22 |
| Test | 23–26 |

The baseline model is a **logistic regression** using transaction amount, relative time, card attributes, address fields, anonymized product category, email domains, and match indicators.

Numeric missing values are median-imputed. Categorical variables are imputed and one-hot encoded.

Because the model uses balanced class weights, I treat its output as a **risk-ranking score**, not as a calibrated probability of fraud.

On the validation period, the model achieved:

- **ROC-AUC:** 0.780
- **PR-AUC:** 0.140
- **Fraud prevalence:** 0.034

More importantly for this project, observed fraud becomes substantially more concentrated as the model score increases.

The highest risk decile had an observed fraud rate of about **14.9%**, compared with about **0.6%** in the lowest risk decile, or roughly **25.8×** higher.

That gave me a useful ranking signal to move from model evaluation to the actual decision problem.

---

## 3. From Risk Score to Control Decision

A fraud model does not make an operational decision until a threshold is applied.

For each candidate threshold, I simulate what would happen if transactions with model scores above that threshold were flagged.

I track:

| Metric | Question |
|---|---|
| Fraud Capture Rate | How much observed fraud would be flagged? |
| False Positive Rate | How much legitimate activity would also be flagged? |
| Precision | How much of the flagged activity is actually fraud? |
| Fraud Caught | How many fraud transactions would be captured? |
| Fraud Missed | How many fraud transactions would remain below the threshold? |
| Legitimate Interrupted | How many legitimate transactions would be flagged? |
| Value Exposure | How much transaction value sits inside each outcome? |

The pattern is consistent:

**Lower threshold → more fraud captured, but more legitimate activity interrupted.**

**Higher threshold → less legitimate interruption, but more fraud missed.**

For example, in the validation analysis:

| Threshold | Capture | False Positive Rate | Precision | Fraud Caught | Fraud Missed | Legitimate Interrupted |
|---:|---:|---:|---:|---:|---:|---:|
| 0.70 | 54.0% | 14.4% | 11.6% | 1,423 | 1,214 | 10,819 |
| 0.75 | 46.3% | 10.2% | 13.7% | 1,220 | 1,417 | 7,672 |
| 0.85 | 28.5% | 3.9% | 20.4% | 751 | 1,886 | 2,939 |

These scenarios are **not recommendations for an optimal threshold**. They illustrate how quickly the operating outcome changes when the control becomes more or less aggressive.

---

## Fraud Control Center

<img width="1371" height="768" alt="02 Fraud Control Center" src="https://github.com/user-attachments/assets/a8c18759-8cc4-4f00-88f0-cf2cc05d6d72" />


The second dashboard turns the model output into a control simulation.

Instead of displaying a single classification result, it allows the threshold range to be changed and compares:

- fraud capture
- false positive exposure
- precision
- fraud caught and missed
- legitimate activity interrupted
- product-level control behavior
- transaction value affected
- performance across candidate thresholds

This shifts the question from:

> **How accurate is the fraud model?**

to:

> **What happens when we actually act on its risk signal?**

---

## Key Findings

**1. Fraud pressure is not uniform.**  
Observed fraud rates vary meaningfully across relative time, anonymized product categories, payment characteristics, and transaction size.

**2. Portfolio averages hide useful structure.**  
A 3.50% overall fraud rate does not describe the higher-risk pockets visible within the transaction environment.

**3. The model produces useful risk concentration.**  
Observed fraud in the highest risk decile is roughly 25.8× the rate in the lowest risk decile on the validation period.

**4. Better fraud capture creates a control trade-off.**  
Moving the threshold to capture more fraud also increases the amount of legitimate activity flagged.

**5. There is no defensible universal threshold in this dataset.**  
Choosing one would require information about fraud losses, intervention costs, review capacity, customer impact, and the consequences of false positives.

---

## Decision Takeaway

The project does not recommend replacing a fraud model or selecting one universal threshold.

Instead, it supports a different way of evaluating fraud controls:

### Monitor the environment

Fraud exposure changes over time and across transaction groups, so portfolio averages should not be the only monitoring layer.

### Separate ranking from decision policy

A useful risk score does not automatically determine where the control threshold should sit.

### Evaluate both sides of the intervention

Fraud capture should be considered alongside legitimate transaction friction and transaction value affected.

### Test whether one threshold is enough

Product and temporal differences make it worth asking whether a single global control behaves consistently across the transaction environment.

The main analytical shift is simple:

> **The problem is not only how much fraud a model can detect. It is how aggressively a control should act on that signal when stronger intervention also affects legitimate activity.**

---

## What the Data Does Not Tell Us

Several limitations are intentionally preserved rather than filled with assumptions.

- The dataset does not contain actual approve, decline, block, challenge, or manual-review decisions.
- **Legitimate interrupted** is simulated friction, not observed customer experience.
- A flagged legitimate transaction cannot be interpreted as an actual customer decline or complaint.
- Transaction value associated with fraud is not verified financial loss.
- The currency of `TransactionAmt` is not established in this analysis.
- `ProductCD` is anonymized and cannot be assigned unsupported business meanings.
- `TransactionDT` provides relative ordering, not real calendar dates.
- Identity information covers only about 24% of transactions and is not evenly distributed across product groups.
- The model risk score is used for ranking and is not presented as a calibrated fraud probability.
- Historical model performance does not establish production performance on future transactions.
- The analysis identifies associations and decision trade-offs, not causal drivers of fraud.

---

## Repository Structure

```text
fraud-control-threshold-analytics/
│
├── data/
│   └── data_source.md
│
├── notebooks/
│   ├── 01_data_audit.ipynb
│   └── 02_risk_model.ipynb
│
├── visuals/
│   ├── 01_transaction_risk_monitor.png
│   └── 02_fraud_control_center.png
│
├── .gitignore
├── LICENSE
└── README.md
```

### Notebooks

[`01_data_audit.ipynb`](notebooks/01_data_audit.ipynb)  
Audits the transaction environment and examines descriptive fraud patterns across time, transaction characteristics, identity coverage, products, and value exposure.

[`02_risk_model.ipynb`](notebooks/02_risk_model.ipynb)  
Builds the temporal risk model, evaluates risk concentration, and simulates the trade-off created by different fraud-control thresholds.

---

## Tools

**Python**  
Pandas, NumPy, scikit-learn, Matplotlib

**Analytics**  
Exploratory data analysis, temporal validation, fraud risk ranking, threshold analysis, class-imbalance evaluation

**Visualization**  
Power BI

---

## Reproducing the Analysis

The raw IEEE-CIS competition files are not redistributed in this repository.

To reproduce the project:

1. Obtain `train_transaction.csv` and `train_identity.csv` from the IEEE-CIS Fraud Detection competition.
2. Store the files locally under `data/raw/`.
3. Run `01_data_audit.ipynb`.
4. Run `02_risk_model.ipynb`.

See [`data/data_source.md`](data/data_source.md) for additional dataset and provenance notes.

---

## Why I Built This

I wanted to build a fraud analytics project where the model was not the endpoint.

The more interesting question to me was what happens after a model produces a risk score. Once that score controls whether a transaction is flagged, model performance becomes connected to customer friction, transaction value, and operating decisions.

That shift from **prediction → decision** became the focus of the project.
