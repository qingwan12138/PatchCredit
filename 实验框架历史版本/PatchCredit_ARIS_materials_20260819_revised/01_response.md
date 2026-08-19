# Independent novelty audit response — revised PatchCredit

> As of 2026-08-19. Verdict vocabulary: `direct`, `partial`, `novel/not found`.  
> Overall verdict: **PatchCredit revised = partial but defensible combined boundary**. The revision is materially safer than the 2026-08-18 version because patch minimization is no longer treated as the core contribution.

## 1. Critical change after new competitor check

The strongest newly added collision is **TRIM: Reducing AI-Generated CodeSlop via Agent Trajectory Minimization** (2026-07-20, arXiv:2607.18161). TRIM already performs hierarchical counterfactual removal of agent-generated edits, validates removals through execution, and minimizes patches more efficiently than Delta Debugging. Therefore:

- “agent patch → delete edits → validate → smaller patch” cannot be the PatchCredit novelty;
- dependency-aware or hierarchical patch minimization alone is insufficient;
- PatchCredit must instead center on **runtime performance credit and edit interaction structure**.

TRIM: https://arxiv.org/abs/2607.18161

## 2. Revised claim-level verdict

| Claim | Verdict | Closest prior art | Defensible boundary / kill condition |
|---|---|---|---|
| A1: edit-level runtime credit + interaction inside LLM performance patches | **partial; combined boundary not found** | ECCO; Muppet; Agents that Matter; TRIM | Defensible only if the system measures performance contributions of dependency-closed edits and explicitly models synergy/conflict under runtime noise. Kill if LOO/simple regression provides equally stable decisions at equal budget. |
| A2: budgeted counterfactual subset selection | **partial** | delta debugging; active subset selection; attribution literature | Must show fewer executions for the same recomposition quality / interaction recovery. Algorithmic novelty alone should not be overclaimed. |
| A3: performance-guided recomposition | **partial; stronger than minimization framing** | Muppet; TRIM; generic search | Publishable value comes from using measured credit/interactions to find a better correctness-preserving edit combination. “Smaller patch” alone is not enough. Kill if recomposition merely reproduces ddmin/LOO results. |
| A4: LLM semantic unitization / interaction hypotheses | **partial and risky** | SemOpt; compiler-agent systems | Must ablate against AST/def-use only grouping and non-LLM interaction models. If the LLM adds no measurable search or generalization benefit, describe the work as analysis of LLM-generated patches rather than an LLM method. |

## 3. Important near-neighbors

- **ECCO** studies evidence-driven reasoning and performance effects for compiler pass-sequence optimization; it makes “performance evidence / synergy” non-novel as a general concept, but the object is pass sequencing rather than heterogeneous edits inside a generated patch.  
  https://arxiv.org/abs/2602.00087
- **Muppet** already searches correctness-preserving mutation subsets for speed; subset pruning for performance is therefore not new by itself.  
  https://doi.org/10.1016/j.parco.2024.103097
- **SWE-Pro** evaluates repository-level performance patches with parameterized, noise-aware runtime/memory tests; multi-input/noise-aware patch evaluation cannot be claimed as novelty.  
  https://arxiv.org/abs/2606.25530
- **Benchmark reliability audit** shows cross-machine/runtime instability can dominate performance conclusions, making repeated measurement and confidence bounds mandatory.  
  https://arxiv.org/abs/2607.01211
- **SemOpt** mines, describes, clusters and reuses optimization strategies from historical changes; therefore “Motif Distiller” should stay an optional extension, not a thesis contribution.  
  https://arxiv.org/abs/2510.16384

## 4. Recommended thesis contribution structure

1. **Main innovation — Dependency-aware Performance Credit & Interaction Modeling**  
   Estimate each edit unit's runtime contribution and pairwise/higher-order synergy/conflict using correctness-gated, noise-controlled counterfactual execution.
2. **Support innovation 1 — Budgeted Interaction Sampling**  
   Select the most informative valid subsets instead of enumerating all `2^n` combinations.
3. **Support innovation 2 — Performance-guided Patch Recomposition**  
   Search for a correctness-preserving combination that maximizes measured speedup subject to complexity/measurement constraints; it may be smaller than or faster than the original full patch.

## 5. Recommended wording

Safe one-sentence claim:

> PatchCredit analyzes a successful LLM-generated multi-edit optimization patch by measuring dependency-aware edit-level runtime credit and edit interactions, then uses the measured structure to recompose a correctness-preserving patch with equal or better runtime performance under a limited measurement budget.

Avoid:

> We are the first to minimize LLM patches with counterfactual deletion.

## 6. Pilot gate

Proceed only if all of the following can be demonstrated on a small set of patches:

- at least 4–8 meaningful dependency-closed edit units exist per selected patch;
- repeated runtime sessions produce stable full-patch speedups and sufficiently stable subset comparisons;
- non-additive interactions occur often enough to matter;
- budgeted sampling beats or matches exhaustive/LOO quality with materially fewer runs;
- recomposition produces equal-or-better speedup than the full patch on a meaningful subset of tasks, or achieves a clear speedup/complexity Pareto improvement;
- LLM-aided grouping/hypothesis generation adds value over static-only grouping.

## 7. CostWitness

Remain **conditional backup only**. The previous concern is unchanged: VPlan and backend-informed cost modeling cover much of its core. Do not implement it in parallel with PatchCredit unless a small data audit first shows a real runtime-misranking signal that simple calibration/GBDT/bandit cannot solve.
