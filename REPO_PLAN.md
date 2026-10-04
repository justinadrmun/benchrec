# BenchRec repo plan

## Goal

Build a modular BenchRec research repo for graph-based record linkage and link prediction. The system should support:

- preprocessing and schema validation
- graph construction from transaction and attribute nodes
- graph embedding with Attri2Vec-style random-walk training
- negative sampling strategies
- inductive link prediction on unseen nodes
- optional reranking
- Top Boundary Ranking (TBR) post-processing for group record linkage
- sequential validation before final eval on Kaggle files
- analysis of error patterns to guide iterative improvement

## Core structure

```txt
benchrec/
  README.md
  CONTEXT.md
  COMPETITION.md
  DATA_DICTIONARY.md
  REPO_PLAN.md

  pyproject.toml
  justfile

  configs/
    base.yaml
    data/
      benchrec.yaml
    graph/
      default.yaml
    embedding/
      attri2vec.yaml
    negatives/
      hard.yaml
    postprocess/
      tbr.yaml
    eval/
      kaggle.yaml
    research/
      default.yaml

  src/benchrec/
    cli.py
    config.py

    data/
      download.py
      load.py
      schema.py
      splits.py
      validation.py

    graph/
      builder.py
      features.py
      node_types.py
      bipartite.py

    embedding/
      model.py
      trainer.py
      losses.py
      checkpoints.py

    sampling/
      negatives.py
      hard_negative.py
      mining.py

    linkpred/
      scorer.py
      trainer.py
      reranker.py

    postprocess/
      ranking.py
      grouping.py
      one_to_one.py
      one_to_many.py
      assignment.py

    eval/
      kaggle_metric.py
      local_metric.py
      analyze_errors.py
      report.py

    pipelines/
      preprocess.py
      train_embedding.py
      mine_negatives.py
      train_linkpred.py
      predict.py
      postprocess.py
      evaluate.py
      analyze.py

    research/
      harness.py
      strategy.py

  research/
    program.md
    results.tsv
    runs/

  tests/
    test_schema.py
    test_metric.py
    test_postprocess.py

  notebooks/
    EDA.ipynb
    experiments.ipynb
```

## Pipeline phases

1. Preprocessing
   - load train/eval/solution CSVs
   - validate schema and types
   - choose attribute columns to represent nodes
   - convert to canonical transaction representations
   - create sequential train/val split for local tuning

2. Graph construction
   - build bipartite graph of A-side and B-side transactions
   - optionally add attribute nodes and their edges
   - define edge features / node features
   - create graph datasets for train and eval

3. Embedding / random walk training
   - train Attri2Vec-style embeddings
   - use negative sampling and random walks
   - store checkpointed embeddings for downstream use

4. Hard negative mining
   - mine challenging false-positive candidate pairs
   - rebalance training/evaluation data
   - make training more robust to ambiguous matches

5. Link prediction and reranking
   - generate candidate edges between B nodes and A nodes
   - score edges using learned embeddings and/or tabular features
   - optional reranker to refine top candidates

6. Top Boundary Ranking (TBR)
   - apply group-record-linkage logic
   - enforce one-to-one or one-to-many matching rules
   - threshold candidate sets for final business-meaningful allocations

7. Evaluation
   - local sequential tuning on train/val
   - final evaluation on eval + solution CSVs
   - precision gate first, then maximize match rate

8. Error analysis
   - inspect false positives/negatives by amount, account, date, rule, etc.
   - identify weak attribute representations or graph construction issues

## Research loop

This repo should support a Karpathy-style autoresearch loop, following the pattern from https://github.com/karpathy/autoresearch:

- a fixed harness computes the objective and prints a stable metric
- one or a few mutable strategy files are tuned by agents
- experiments are kept/discarded based on objective improvement
- the human can change data representation, embedding method, negative sampling, or postprocessing logic without reworking the entire project
- the loop is meant to operate on top of a clean, modular codebase rather than a monolithic notebook

## Key design principle

The project is not a generic tabular ML repo. It is a structured research system for graph-based reconciliation. The architecture must keep the experiment surface narrow but flexible:

- stable data and eval interfaces
- editable strategy logic for graph construction and model/hparam changes
- reusable postprocessing and analysis modules
