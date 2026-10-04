# BenchRec (benchmarkteam) — Evaluation + Required Datasets

## Task + evaluation

> Given a transaction of type B (external statement entry), predict the key characteristics of the transaction(s) of type A (internal ledger entries) that it should match to.  
> Instead of predicting specific transaction IDs, the model predicts a business-meaningful allocation key:  
> A_allocation = currency + account + transaction attributes

> A prediction is considered correct if the predicted allocation set is:  
> - equal to, or  
> - a subset of the true allocation set

> In production, erroneous matches are costly. It is better to leave a transaction unmatched (and leave it for manual review) than to match it incorrectly. The required precision for fully automated matches it 99.9%. However for this dataset this required precision has been reduced to 99.8% in order to account for labels errors which were identified at a rate of 0.2%. The match rate should be optimized subject to the match precision meeting the required level.

> Match Rate  
> Fraction of true matches your model successfully identifies.

> Match Precision  
> Fraction of your predicted matches that are correct.

---

## Required datasets/files (found in dataset file listing)

From `datasets/list`:
- `BenchRec_cash_v1.0_train.csv` (78,765,900 bytes; n=149,855)
- `BenchRec_cash_v1.0_eval.csv` (25,850,839 bytes; n=69,172)
- `BenchRec_cash_v1.0_solution.csv` (6,715,938 bytes; n=32,049)
- `MatcherByChatGPT_submission.csv` (10,537,973 bytes) *(extra example submission file)*

## Some logic found in the evaluation notebook code

The Kaggle notebook code (`calculatematchrateandprecisionv1-1`) includes this metric logic:

```python
# Validate required columns (case-insensitive for solution)
required_sub = ["b_id", "targetallocation"]
required_sol = ["b_id", "targetallocation"]

# Correctness rule: predicted allocation is correct if pred_set ⊆ true_set (and pred_set non-empty)
df["subset_correct"] = df.apply(lambda r: (len(r["pred_set"]) > 0) and r["pred_set"].issubset(r["true_set"]), axis=1)

match_rate = matched / total if total else np.nan
subset_acc_matched_only = subset_ok / matched if matched else np.nan

# Define the STP precision requirement
stpActualPrecisionTarget = 0.999
estimatedLabelErrorRate = 0.002
achievableStpprecisionTarget = (stpActualprecisionTarget - estimatedLabelErrorRate)

if ( subset_acc_matched_only > achievableStpprecisionTarget ):
    print( "precision requirement met")
    print(f"Match rate: {match_rate:.4%}")
else:
    print( "precision requirement not met")
```

Also found in the same code comments:

> i.e. the predicted match members are a subset of the labelled match members.

---

## Practical takeaway for model building

- Predict `targetAllocation` for each `B_id` in eval/submission.
- Your predicted set must be a **subset** of true allocation set to be counted correct.
- Precision is a hard gate first; then optimize match rate.
- Precision threshold target described in source text: effectively **99.8%** (99.9% STP requirement adjusted by 0.2% estimated label error).

---

## Sources used
- Dataset metadata API:
  - https://www.kaggle.com/api/v1/datasets/view/benchmarkteam/benchrec-real-world-cash-reconciliation-dataset
- Dataset files list API:
  - https://www.kaggle.com/api/v1/datasets/list/benchmarkteam/benchrec-real-world-cash-reconciliation-dataset
- Evaluation notebook API (Kaggle code):
  - https://www.kaggle.com/api/v1/kernels/pull/benchmarkteam/calculatematchrateandprecisionv1-1
- Dataset download endpoints (for headers/shape checks):
  - https://www.kaggle.com/api/v1/datasets/download/benchmarkteam/benchrec-real-world-cash-reconciliation-dataset/BenchRec_cash_v1.0_train.csv
  - https://www.kaggle.com/api/v1/datasets/download/benchmarkteam/benchrec-real-world-cash-reconciliation-dataset/BenchRec_cash_v1.0_eval.csv
  - https://www.kaggle.com/api/v1/datasets/download/benchmarkteam/benchrec-real-world-cash-reconciliation-dataset/BenchRec_cash_v1.0_solution.csv
