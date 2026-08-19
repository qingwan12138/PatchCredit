# ARIS Output Manifest

## PatchCredit Revision — 2026-08-19 / run `20260819_123200`

| Artifact | Fixed path | Versioned path | Status |
|---|---|---|---|
| 修订后的 PatchCredit 研究框架 | `IDEA_REPORT.md` | `IDEA_REPORT_20260819_123200.md` | Main innovation changed to performance credit + interaction; pilot not run |
| 修订后的竞品/碰撞矩阵 | `COMPETITOR_MATRIX.md` | `COMPETITOR_MATRIX_20260819_123200.md` | Added TRIM; downgraded minimization; Motif Distiller demoted |
| Novelty re-audit request | `01_request.md` | — | Revised claims A1–A4 |
| Novelty re-audit response | `01_response.md` | — | PatchCredit revised: partial but defensible combined boundary |
| Run metadata | `run.meta.json` | — | Literature-screened; no experiment results claimed |

## Major differences from 2026-08-18 package

1. **TRIM added as a strongest direct near-neighbor** for agent patch minimization.
2. Patch minimization is no longer a formal innovation.
3. Main innovation is now **Dependency-aware Performance Credit & Interaction Modeling**.
4. Support innovation 1 is **Budgeted Interaction Sampling**.
5. Support innovation 2 is **Performance-guided Patch Recomposition**.
6. The optimization objective now prefers equal-or-better runtime than the full LLM patch; patch size is a secondary regularizer/Pareto objective.
7. `Motif Distiller` is moved to optional extension because strategy extraction/reuse is already crowded (e.g. SemOpt).
8. CostWitness remains a conditional backup only and should not be implemented in parallel.

## Recommended next gate

Run a small exhaustive pilot first: 10–15 correct, significantly faster LLM multi-edit patches with 4–8 dependency-closed units. Verify that non-additive interactions and recomposition opportunities actually exist before implementing a sophisticated sampler.

No PatchCredit experiment has been executed in this package.
