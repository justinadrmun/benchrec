# BenchRec repo context

This repository is a research scaffold for the BenchRec reconciliation benchmark. It is not a generic Kaggle notebook. The goal is to support a structured graph-based matching pipeline that can evolve over many experiments in a Karpathy-style autoresearch loop, mirroring the design used in the reference project at https://github.com/karpathy/autoresearch.

## Reference documents

Use these files as the canonical background:

- `COMPETITION.md` — the benchmark goal, required datasets, and evaluation metric.
- `DATA_DICTIONARY.md` — the actual CSV schema and what each field means.
- `REPO_PLAN.md` — the project layout and the intended modular pipeline.

These are the "ground truth" files for later agents. They describe the benchmark problem, the scoring logic, and how to structure the engineering work within an autoresearch-oriented workflow modeled on that reference repository.

## Core task

BenchRec is a record-linkage / reconciliation problem:

- B-side statement transactions must be matched to internal A-side ledger entries.
- The target is not a direct transaction ID.
- The model predicts an allocation key in `targetAllocation`.
- Evaluation is precision-gated first, then optimized for match rate.

The practical modeling objective is therefore:

- produce high-quality candidate links
- keep false matches below the precision threshold
- optimize match coverage under that constraint

## Proposed approach

The working idea for this repo is a graph-based pipeline:

1. Preprocess the CSV data into canonical transaction representations.
2. Select attribute columns to create node types and graph structure.
3. Build a bipartite graph between A-side and B-side records (optionally with attribute nodes).
4. Train a graph embedding model on random walks / neighborhood structure.
5. Use hard negative sampling to improve discrimination between close matches and ambiguous non-matches.
6. Score candidate links using inductive link prediction on unseen nodes.
7. Optional reranking to refine candidate edges.
8. Apply Top Boundary Ranking (TBR) for grouping / allocation postprocessing.
9. Evaluate on a local sequential split before using the eval + solution files.
10. Analyse errors to improve representation and graph design.

## Why this matters

This setup intentionally separates concerns:

- data preprocessing and schema handling are stable
- graph construction is flexible and experiment-driven
- embedding training is isolated from postprocessing
- hard negative sampling and reranking are independent optimization stages
- TBR is applied at the final decision layer where record-linkage constraints matter

This is the right pattern for iterative research with agents and structured experimentation. In the autoresearch pattern from https://github.com/karpathy/autoresearch, the agent keeps the data/evaluation harness stable and only mutates the experimental code that changes graph construction, embeddings, sampling, postprocessing, or ranking logic.

## How agents should use this repo

Agents should treat this repo as a structured experimental system, not as a one-off notebook.

- Do not rewrite the benchmark understanding in ad hoc ways.
- Keep the evaluation logic fixed and transparent.
- Make changes in a small, modular surface area.
- Prefer stable interfaces for preprocessing, graph building, modeling, and evaluation.
- Use sequential validation before final Kaggle evaluation.
- Keep experiment logs and objective metrics comparable across runs.
- The autoresearch loop should be built on top of this structure, not replace it.

## Short rule of thumb

If an experiment changes data representation, graph construction, sampling, model hparams, or postprocessing, it should be isolated in its own module or config, not embedded directly into a notebook or a single giant script.

This repo exists to make those experiments clean, reproducible, and reviewable.
