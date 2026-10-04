# BenchRec

Research scaffold for the BenchRec reconciliation benchmark.

This repository is designed to support a Karpathy-style autoresearch workflow, following the pattern used in the reference project at https://github.com/karpathy/autoresearch: an autonomous agent iterates on a small, well-defined experimental surface, runs experiments, evaluates them against a fixed metric, and keeps or discards changes based on objective evidence.

Core Scientific Workflow:
- preprocess / validate data
- build graph representation from A/B records and attribute nodes
- train graph embeddings with random-walk / Attri2Vec-style methods
- mine hard negatives
- score inductive candidate links
- optional reranking and Top Boundary Ranking (TBR) postprocessing
- evaluate locally on a sequential split
- validate on eval + solution files once the local pipeline is stable

This repo is not just a notebook. It is intended to be a minimal, structured experimental system where the agent can safely edit the mutable research code while the fixed evaluation harness remains stable.

## Karpathy autoresearch model

The reference implementation is the project at https://github.com/karpathy/autoresearch. The canonical idea is:
- the project is intentionally small and structurally constrained
- one or a few files hold the mutable research code
- a fixed harness handles data, evaluation, and scoring
- the agent experiments, logs the result, and keeps only improvements

This repository is meant to follow that same pattern, adapted to the BenchRec graph-linkage task.

The reference pattern is:

1. Keep data loading, preprocessing, and evaluation fixed.
2. Allow experimentation in a small set of mutable files.
3. Run a short fixed-budget experiment.
4. Record a scalar objective.
5. Keep the change if it improves the metric.
6. Revert or discard if it does not.
7. Repeat continuously.

In other words, the agent is not rewriting the project structure each time. It modifies the experiment code, runs it, compares metrics, and moves forward in a controlled loop.

## Reference docs

- `COMPETITION.md` — benchmark goal and metric
- `DATA_DICTIONARY.md` — field-level schema
- `CONTEXT.md` — repo intent and expected workflow
- `REPO_PLAN.md` — modular project structure

## Repo goal

The goal is to make BenchRec research easy to automate and compare:
- fixed data contracts
- stable evaluation logic
- modular graph/embedding/postprocess stages
- a narrow mutation surface for agents
- compatibility with an autoresearch-style loop

## Typical workflow

1. preprocess / validate CSVs
2. build graph representation
3. train embeddings with Attri2Vec-like graph learning
4. mine hard negatives
5. score candidate links and rerank
6. run TBR / group matching postprocess
7. evaluate locally on a sequential split
8. test against eval + solution files when ready

## Local commands

See `justfile` for task shortcuts.