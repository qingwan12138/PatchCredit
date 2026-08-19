# Novelty-check request — PatchCredit revised

> Re-audit date: 2026-08-19  
> Purpose: re-check the revised PatchCredit after demoting patch minimization and promoting runtime performance credit + interaction modeling.

## PatchCredit revised claims

- **A1 — Main claim:** For an organically generated, correctness-passing, speedup-producing LLM multi-edit optimization patch, estimate **edit-level runtime performance credit** and **non-additive interactions** (synergy/conflict) under dependency-closed edit units and noise-controlled execution.
- **A2 — Support claim 1:** Use dependency information, current uncertainty, and observed interaction structure to perform **budgeted counterfactual subset selection**, reducing the number of compile/test/run experiments compared with exhaustive enumeration while preserving useful credit/interaction estimates.
- **A3 — Support claim 2:** Use measured credit and interactions for **performance-guided recomposition**, targeting the best correctness-preserving subset/combination; the goal is not merely a smaller patch and may exceed the full LLM patch's speedup by removing negative or conflicting edits.
- **A4 — LLM role:** Use the LLM only for semantic edit-unit proposals and interaction hypotheses; all legality, correctness and performance claims must be verified by compiler/static checks, tests, and runtime measurement.

## Claims explicitly removed or downgraded

- “Counterfactual patch deletion/minimization” is **not** a standalone novelty claim.
- “Retain 95% speedup while deleting 30% edits” is an evaluation gate, not novelty.
- “Motif Distiller / optimization-pattern reuse” is an optional extension, not a formal thesis innovation.
- Do not use strict causal language unless identifiability assumptions are justified; default term is `performance credit / counterfactual contribution estimate`.

## Required comparisons

TRIM (2026), ECCO (2026), Muppet (2024), SWE-Pro (2026), SWE-Perf/GSO/SWE-efficiency reliability audit (2026), Agents that Matter (2026), SemOpt (2025/2026), SBLLM, delta debugging, LOO, Shapley approximation, sparse regression/GBDT interaction models.

## CostWitness

Keep CostWitness only as a conditional backup. Reuse the previous audit: backend-informed costing is prior art; only structured disagreement activation + typed witness + sparse measurement may remain, and only if simple calibration/GBDT/bandit baselines fail.
