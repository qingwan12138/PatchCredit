# 竞品矩阵：PatchCredit 修订版（性能贡献与交互归因）

> 检索/修订日期：2026-08-19  
> ARIS 阶段：`research-lit → idea-creator → novelty-check → revision`  
> 状态：**完成第二轮碰撞审计；尚未运行 pilot；不声称已有性能提升**

## 1. 修订后的查新口径

PatchCredit 不再把“删除冗余 patch / 得到最小 patch”作为主创新，而把研究对象收紧为：

> **一个已经正确且实测加速的 LLM 多编辑优化 patch 中，每个编辑对运行时性能贡献多少、哪些编辑之间存在协同或冲突，以及如何在有限实测预算下利用这些信息重组出更优 patch。**

因此竞品比较分为五类：

1. agent patch minimization；
2. 性能贡献/交互归因；
3. 性能 patch / code optimization benchmark；
4. 性能导向 mutation/subset search；
5. optimization strategy mining/reuse。

## 2. 竞品总表

| 工作 | 年份 | 核心对象/机制 | 与 PatchCredit 修订版的关系 | 必须避免的错误主张 | 当前结论 |
|---|---:|---|---|---|---|
| [TRIM](https://arxiv.org/abs/2607.18161) | 2026 | agent trajectory/patch；层次化反事实删除 + 执行验证 + 最小化 | **最强新增近邻**。已经直接覆盖“agent patch 反事实删除/最小化” | 不能再说“首次通过反事实删除精简 LLM patch” | Patch minimization 降级为结果，不做主创新 |
| [ECCO](https://arxiv.org/abs/2602.00087) | 2026 | compiler pass sequence；性能证据、pass 贡献/协同、LLM+GA | “性能 evidence / synergy”概念层面强近邻，但粒度是 pass 序列 | 不能说“首次发现优化单元协同” | PatchCredit 必须突出 heterogeneous edit + runtime counterfactual + recomposition |
| [Muppet](https://doi.org/10.1016/j.parco.2024.103097) | 2024 | AST/OpenMP mutation subset；正确性保持下的性能搜索 | 已覆盖性能导向编辑子集搜索 | 不能把“删编辑后保留/提高速度”作为独立创新 | 需要 edit credit + interaction + budgeted measurement 的联合闭环 |
| [Agents that Matter](https://arxiv.org/abs/2605.27621) | 2026 | agent contribution attribution；LOO/组合归因 | 说明 attribution 与 LOO 强基线不可忽略 | dependency-aware attribution 本身不够新 | 必须做 equal-budget LOO / Shapley / regression 对比 |
| [SWE-Pro](https://arxiv.org/abs/2606.25530) | 2026 | 102 个真实性能优化任务；参数化 runtime/memory；噪声感知 | 可作外部任务来源/评测框架；整 patch 评分强近邻 | 不能把“多输入、噪声感知性能测试”当创新 | 可用于 benchmark/harness，不覆盖 patch 内 credit |
| [SWE-Perf](https://arxiv.org/abs/2507.12415) | 2025 | 140 个真实仓库 performance PR 任务 | 真实 performance patch 来源 | 不应声称真实仓库性能 patch benchmark 是新贡献 | 可作候选 patch/data source |
| [Performance benchmark reliability audit](https://arxiv.org/abs/2607.01211) | 2026 | GSO/SWE-Perf/SWE-efficiency 跨机器复现 | 直接约束 PatchCredit 的统计可信度 | 不能用单次 runtime 判定 credit | 必须重复测量、CI/显著性、机器固定化 |
| [SemOpt](https://arxiv.org/abs/2510.16384) | 2025/2026 | 从历史优化修改中提取、描述、聚类、复用 optimization strategy | 与原 Motif Distiller 高度接近 | 不能把“从 patch 抽取并复用优化 motif”当正式创新 | Motif Distiller 降为 optional extension |
| [SBLLM](https://arxiv.org/abs/2408.12159) | 2024 | LLM code optimization / search | 整体候选生成与优化近邻 | 不能把“生成多个优化候选再测试”当创新 | PatchCredit 是 post-hoc analysis + recomposition |
| Delta Debugging | 经典 | 通过删除变更最小化 failure-inducing set | 直接算法基线 | “最小 patch”不新 | ddmin 必须作为强基线 |
| LOO / Shapley / sparse regression / GBDT interaction | 经典/通用 | 贡献归因与交互估计 | 非 LLM 强基线 | 不能只与 random 比 | 必须 equal-budget 比较稳定性与重组结果 |

## 3. 修订后的不可替代边界

PatchCredit 至少要同时满足下面五点，才不会退化成 TRIM、ECCO、Muppet 或普通 attribution：

1. **对象特殊：** 一个已经正确、已有实测 speedup 的自然 LLM 多编辑优化 patch；
2. **单元合法：** 编辑单元由 AST/def-use/control dependency/必要声明形成 dependency-closed units，而不是任意 diff hunk；
3. **性能而非功能 credit：** 每个可行子集先过编译/测试，再做真实 runtime 测量；
4. **显式 interaction：** 不只做单编辑 LOO，而是识别 synergy/conflict，并量化不确定性；
5. **用 credit 做决策：** 不是为了写解释，而是用 credit/interaction 指导新的 patch recomposition，并比较最终 runtime。

其中第 5 点很关键：如果 credit 只生成一张“解释图”，而没有改善重组/搜索结果，论文价值会明显下降。

## 4. 三个正式创新点的竞品压力

### 4.1 主创新：Dependency-aware Performance Credit & Interaction

**目标：** 对多编辑优化 patch 建立运行时贡献结构。

需要回答：

- 单个编辑的边际性能贡献是多少？
- 编辑 A、B 单独收益小，但 `A+B` 是否有明显协同？
- 某编辑是否与其他编辑冲突，导致完整 patch 比某个子集更慢？
- 贡献排序在重复测量会话中是否稳定？

**最强压力：** ECCO（性能证据/协同）、Agents that Matter（贡献归因）、Muppet（性能 subset）。

**可守边界：** heterogeneous LLM patch edits + dependency closure + correctness-gated runtime counterfactual + interaction + recomposition。

### 4.2 支撑创新 1：Budgeted Interaction Sampling

当 patch 有 `n` 个单元时，完整组合有 `2^n` 个；10 个单元即 1024 个组合。正式方法不能依赖长期穷举。

采样器可综合：

- dependency closure；
- 当前 credit uncertainty；
- 已观察到的 interaction residual；
- 编译/测试失败概率；
- measurement cost。

**必须比较：** exhaustive（小 patch oracle）、random subset、LOO、ddmin、Shapley sampling、sparse regression/GBDT active sampling。

创新是否成立看的是：**相同重组质量/interaction recovery 下减少多少真实执行预算**，而不是算法名字是否新。

### 4.3 支撑创新 2：Performance-guided Patch Recomposition

原版目标“保留 95% speedup 并删 30% 编辑”降为 baseline/评价门槛。修订目标是：

\[
\max_{S \subseteq P}\; \mathrm{Speedup}(S)-\lambda\,\mathrm{Complexity}(S)
\]

subject to:

\[
\mathrm{Correct}(S)=1
\]

实际可先把 `Complexity(S)` 简化为 edit-unit count 或 diff size。

最有价值的结果不是：

> full patch 1.20× → compact patch 1.19×

而是可能发现：

> full patch 1.20× → 删除负贡献/冲突编辑后 1.27×。

如果大部分任务只能做到“更小但不更快”，仍可作为 Pareto trade-off 结果，但论文主叙事要相应降低。

## 5. Motif Distiller 的处理

原报告把 attribution motif reuse 作为第二阶段贡献。修订后：

- **不作为硕士论文三个正式创新点之一**；
- 只作为 optional extension；
- 必须与 SemOpt 的 strategy extraction/clustering/reuse 明确区分；
- 如果没有 held-out hit-rate/search-cost 改善，直接删除这一模块，不影响主框架成立。

## 6. Benchmark / 数据来源建议

优先级建议：

1. **SWE-Pro**：102 个真实专家优化任务，带参数化性能测试；适合真实仓库外验；
2. **SWE-Perf**：140 个真实 performance PR 任务；适合 patch/data source；
3. **SemOpt C/C++ tasks**：151 个优化任务，可用于生成更多候选 LLM patch；
4. **自生成成功 patch pool**：对 PolyBench/C、LLVM test-suite、性能热点使用多个 LLM/agent 生成 patch，只保留 `correct + statistically significant speedup + multi-edit` 的样本。

注意：SWE-Pro 论文显示现有 LLM 可靠 runtime gain 很少，因此不能假设直接拿到大量“成功 LLM patch”；需要“真实专家 patch + LLM 生成 patch + 可控人工 interaction patch”三类数据共同支撑。

## 7. 关键实验基线

- Full original LLM patch；
- ddmin / TRIM-style minimization；
- LOO；
- random subset；
- approximate Shapley；
- sparse linear / LASSO interaction；
- GBDT interaction；
- exhaustive oracle（仅小 patch）；
- static-only unitization vs LLM-aided semantic unitization。

主要指标：

- final runtime / speedup；
- relative speedup vs full patch；
- patch complexity / edit count；
- interaction recovery（在可控合成或 exhaustive 小 patch 上）；
- measurement budget；
- repeated-session stability；
- correctness pass rate；
- cross-input / held-out input robustness。

## 8. 当前查新结论

| 项目 | 2026-08-18 原版 | 2026-08-19 修订版 |
|---|---|---|
| 核心目标 | attribution + minimal fast patch | **performance credit/interaction + better recomposition** |
| Patch minimization | 核心模块/卖点 | **降级为结果和基线** |
| TRIM | 漏检 | **新增最强直接近邻** |
| Motif Distiller | 第二阶段贡献 | **降为 optional extension** |
| 论文三创新 | 不够集中 | **1 主 + 2 支撑，边界清楚** |
| 最终性能故事 | 尽量保留原 speedup | **争取达到/超过 full LLM patch；否则报告 Pareto trade-off** |
| 当前新颖性判断 | partial | **partial，但联合边界更稳** |

## 9. CostWitness 状态

CostWitness 保留为条件性备选，但不建议与 PatchCredit 同时开发。其 backend-informed costing / legal-plan enumeration 与 LLVM VPlan 和已有 cost-model 工作碰撞较强。只有在先验小实验中证明“structured disagreement → runtime misranking”且 calibration/GBDT/bandit 不足时，才值得恢复。

## 10. 诚信边界

- 当前没有运行任何新性能实验；本文所有数字均为 benchmark 规模、文献结果或未来 gate，不是 PatchCredit 实验结果。
- `llvm-mca`/静态 proxy 只能用于筛选，不能替代最终 runtime。
- performance credit 必须使用重复运行、固定环境和统计置信；不得从一次测量推出“因果”。
- 最终投稿前需再次执行增量查新，尤其检查 2026 年下半年新的 agent patch attribution/minimization 工作。
