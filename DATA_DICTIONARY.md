# BenchRec Data Dictionary

This file summarizes the schema used in the published BenchRec CSVs. The descriptions below are derived from the official dataset task description and the actual column names in the train/eval files.

## 1) Train / Eval file schema

The train and eval files share the same schema. The key task is to predict `targetAllocation` for each `B_id` transaction, where the target describes the business-meaningful allocation of the matching A-side internal transaction(s).

| Field | Description |
| --- | --- |
| `matchId` | Identifier for the matching record / reconciliation event. This groups the related A-side and B-side transaction information for a match candidate. |
| `matchDate` | Date associated with the match or reconciliation decision. |
| `matchRule` | Reconciliation rule name or rule code that was used to create or justify the match (for example, `RULE 4`). |
| `matchedBy` | Source or actor responsible for the match; may be an automated rule, analyst, or system label. |
| `wasPreviouslyMismatched` | Binary flag indicating whether the transaction was previously mismatched. `0` typically means no prior mismatch. |
| `A_transactionType` | Transaction type on the internal A-side ledger. In this dataset, A-side records represent internal ledger entries. |
| `A_id` | Internal ledger transaction identifier on the A side. |
| `A_allocation` | The allocation string already associated with the A-side internal transaction. This is the business-level grouping/label describing how the internal transaction should be allocated. |
| `A_importDate` | Import date for the A-side transaction record. |
| `A_debitOrCredit` | Whether the A-side transaction is a debit or credit (`DR` or `CR`). |
| `A_amount` | Monetary amount for the A-side internal ledger transaction. |
| `A_valueDate` | Date on which the value of the A-side transaction is effective. |
| `A_currencyCode` | Currency code for the A-side transaction (for example, `USD`). |
| `A_account` | Account identifier associated with the A-side ledger transaction. |
| `A_transactionReferences` | Reference text or memo data attached to the A-side transaction, often used for matching and reconciliation. |
| `A_transactionAttributes` | Additional descriptive attributes or free-form metadata for the A-side transaction. This can include narrative information used during reconciliation. |
| `B_transactionType` | Transaction type on the external B-side statement record. In this dataset, B-side records are bank statement transactions. |
| `B_id` | External bank statement transaction identifier on the B side. |
| `B_importDate` | Import date for the B-side bank statement transaction. |
| `B_debitOrCredit` | Whether the B-side statement transaction is a debit or credit (`DR` or `CR`). |
| `B_amount` | Monetary amount of the external bank statement transaction. |
| `B_valueDate` | Value date for the B-side transaction. |
| `B_currencyCode` | Currency code for the B-side statement transaction, such as `USD`. |
| `B_account` | Account identifier associated with the B-side bank statement record. |
| `B_transactionReferences` | Reference text or memo fields attached to the B-side bank statement transaction. |
| `B_transactionAttributes` | Additional descriptive metadata for the B-side bank record, often including free-text or transaction details. |
| `targetAllocation` | Target label for the task. This is the business-meaningful allocation key that the model should predict for the B-side transaction. It is the primary prediction target and represents the correct A-side allocation pattern. |

## 2) What the target means

From the official dataset description:

- The task is to predict the key characteristics of the matching A-side internal transaction(s) for a given B-side statement entry.
- The prediction is not a specific transaction ID.
- Instead, the model predicts a business allocation string such as:
  - `currency + account + transaction attributes`
- The label is stored in `targetAllocation`.
- A prediction is considered correct if the predicted allocation set is equal to, or a subset of, the true allocation set.

This is why the target is a textual allocation key rather than a raw ID match.

## 3) Solution file schema

The solution file is not the training feature matrix. It is used for evaluation and contains the ground-truth labels for the eval set.

| Field | Description |
| --- | --- |
| `B_id` | External bank statement transaction identifier from the B-side data. This is the key used to align submission rows to the solution rows. |
| `targetAllocation` | Ground-truth allocation label for that B transaction. This is what your model is trying to predict. |
| `Usage` | Indicates whether the row is intended for a public/private usage split or similar evaluation partition. In the published file, the values are `Public`. |

## 4) Notes for modeling

- The dataset is a tabular reconciliation / record-linkage problem.
- The model must learn how to infer the correct A-side allocation from B-side context and transaction metadata.
- The prediction target is `targetAllocation`, not a direct transaction match ID.
- Because the scoring logic treats a prediction as correct when the predicted allocation set is a subset of the true allocation set, the problem is structured around allocation correctness rather than exact one-to-one ID matching.

## 5) In plain English

Think of each row as:

- a B-side statement transaction,
- plus its related A-side candidate information,
- plus the label describing the correct allocation on the internal ledger.

Your model is effectively answering:

> “Given the bank statement transaction characteristics, what is the correct internal allocation key for the matching internal ledger record(s)?”
