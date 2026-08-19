# 竞品矩阵：PatchCredit 修订版（性能贡献与交互归因）

> 检索/修订日期：2026-08-19  
> ARIS 阶段：`research-lit → idea-creator → novelty-check → revision`  
> 状态：**完成第二轮碰撞审计；尚未运行 Pilot；不声称已有性能提升**

## 1. 修订后的查新口径

PatchCredit 不再把“删除冗余 Patch / 得到最小 Patch”作为主创新，而把研究对象收紧为：

> **一个已经正确且实测加速的 LLM 多编辑优化 Patch 中，每个编辑对运行时性能贡献多少、哪些编辑之间存在协同或冲突，以及如何在有限实测预算下利用这些信息重组出更优 Patch。**

因此竞品比较分为五类：

1. Agent Patch 最小化；
2. 性能贡献/交互归因；
3. 性能 Patch / 代码优化 Benchmark；
4. 性能导向 Mutation/子集搜索；
5. 优化策略挖掘/复用。

## 2. 竞品总表

| 工作 | 年份 | 核心对象/机制 | 与 PatchCredit 修订版的关系 | 必须避免的错误主张 | 当前结论 |
|---|---:|---|---|---|---|
| [TRIM](https://arxiv.org/abs/2607.18161) | 2026 | agent trajectory/Patch；层次化反事实删除 + 执行验证 + 最小化 | **最强新增近邻**。已经直接覆盖“Agent Patch 反事实删除/最小化” | 不能再说“首次通过反事实删除精简 LLM Patch” | Patch 最小化 降级为结果，不做主创新 |
| [ECCO](https://arxiv.org/abs/2602.00087) | 2026 | 编译器 Pass 序列；性能证据、Pass 贡献/协同、LLM+GA | “性能证据 / 协同”概念层面强近邻，但粒度是 pass 序列 | 不能说“首次发现优化单元协同” | PatchCredit 必须突出 异质编辑 + 运行时反事实测量 + 重组 |
| [Muppet](https://doi.org/10.1016/j.parco.2024.103097) | 2024 | AST/OpenMP Mutation 子集；正确性保持下的性能搜索 | 已覆盖性能导向编辑子集搜索 | 不能把“删编辑后保留/提高速度”作为独立创新 | 需要 编辑贡献 + 交互 + 预算化测量 的联合闭环 |
| [Agents that Matter](https://arxiv.org/abs/2605.27621) | 2026 | Agent 贡献归因；LOO/组合归因 | 说明 归因方法与 LOO 强基线不可忽略 | 依赖感知归因 本身不够新 | 必须做 同预算 LOO / Shapley / 回归模型对比 |
| [SWE-Pro](https://arxiv.org/abs/2606.25530) | 2026 | 102 个真实性能优化任务；参数化 运行时/memory；噪声感知 | 可作外部任务来源/评测框架；整 Patch 评分强近邻 | 不能把“多输入、噪声感知性能测试”当创新 | 可用于 Benchmark/测试框架，不覆盖 Patch 内 贡献 |
| [SWE-Perf](https://arxiv.org/abs/2507.12415) | 2025 | 140 个真实仓库 性能优化 PR 任务 | 真实 性能优化 Patch 来源 | 不应声称真实仓库性能 Patch Benchmark 是新贡献 | 可作候选 Patch/数据来源 |
| [Performance Benchmark reliability audit](https://arxiv.org/abs/2607.01211) | 2026 | GSO/SWE-Perf/SWE-efficiency 跨机器复现 | 直接约束 PatchCredit 的统计可信度 | 不能用单次 运行时 判定 贡献 | 必须重复测量、CI/显著性、机器固定化 |
| [SemOpt](https://arxiv.org/abs/2510.16384) | 2025/2026 | 从历史优化修改中提取、描述、聚类、复用 optimization strategy | 与原 Motif Distiller（优化模式蒸馏器） 高度接近 | 不能把“从 Patch 抽取并复用优化 motif”当正式创新 | Motif Distiller（优化模式蒸馏器） 降为 可选扩展 |
| [SBLLM](https://arxiv.org/abs/2408.12159) | 2024 | LLM 代码优化 / search | 整体候选生成与优化近邻 | 不能把“生成多个优化候选再测试”当创新 | PatchCredit 是 post-hoc analysis + 重组 |
| Delta Debugging | 经典 | 通过删除变更最小化 failure-inducing set | 直接算法基线 | “最小 Patch”不新 | ddmin 必须作为强基线 |
| LOO / Shapley / 稀疏回归 / GBDT 交互模型 | 经典/通用 | 贡献归因与交互估计 | 非 LLM 强基线 | 不能只与 随机方法比较 | 必须 同预算 比较稳定性与重组结果 |

## 3. 修订后的不可替代边界

PatchCredit 至少要同时满足下面五点，才不会退化成 TRIM、ECCO、Muppet 或普通归因方法：

1. **对象特殊：** 一个已经正确、已有实测 加速比 的自然 LLM 多编辑优化 Patch；
2. **单元合法：** 编辑单元由 AST/def-use/控制依赖/必要声明形成 依赖闭合单元，而不是任意 Diff 块；
3. **性能而非功能 贡献：** 每个可行子集先过编译/测试，再做真实 运行时 测量；
4. **显式 交互：** 不只做单编辑 LOO，而是识别 协同/冲突，并量化不确定性；
5. **用 贡献 做决策：** 不是为了写解释，而是用 贡献/交互 指导新的 Patch 重组，并比较最终 运行时。

其中第 5 点很关键：如果 贡献 只生成一张“解释图”，而没有改善重组/搜索结果，论文价值会明显下降。

## 4. 三个正式创新点的竞品压力

### 4.1 主创新：依赖感知的性能贡献与交互建模（Dependency-aware Performance Credit & Interaction）

**目标：** 对多编辑优化 Patch 建立运行时贡献结构。

需要回答：

- 单个编辑的边际性能贡献是多少？
- 编辑 A、B 单独收益小，但 `A+B` 是否有明显协同？
- 某编辑是否与其他编辑冲突，导致完整 Patch 比某个子集更慢？
- 贡献排序在重复测量会话中是否稳定？

**最强压力：** ECCO（性能证据/协同）、Agents that Matter（贡献归因）、Muppet（性能 子集）。

**可守边界：** LLM Patch 内的异质编辑 + 依赖闭合 + 正确性门控 运行时反事实测量 + 交互 + 重组。

### 4.2 支撑创新 1：预算化交互采样（Budgeted Interaction Sampling）

当 Patch 有 `n` 个单元时，完整组合有 `2^n` 个；10 个单元即 1024 个组合。正式方法不能依赖长期穷举。

采样器可综合：

- 依赖闭合；
- 当前 贡献估计不确定性；
- 已观察到的 交互残差；
- 编译/测试失败概率；
- 测量成本。

**必须比较：** 穷举（小 Patch Oracle）、随机子集、LOO、ddmin、Shapley 采样、稀疏回归/GBDT 主动采样。

创新是否成立看的是：**相同重组质量/交互恢复质量 下减少多少真实执行预算**，而不是算法名字是否新。

### 4.3 支撑创新 2：性能引导的 Patch 重组（Performance-guided Patch Recomposition）

原版目标“保留 95% 加速比 并删 30% 编辑”降为 Baseline/评价门槛。修订目标是：

\[
\max_{S \subseteq P}\; \mathrm{加速收益}(S)-\lambda\,\mathrm{复杂度}(S)
\]

subject to:

\[
\mathrm{正确性}(S)=1
\]

实际可先把 `复杂度(S)` 简化为 编辑单元数量 或 diff size。

最有价值的结果不是：

> 完整 Patch 1.20× → compact Patch 1.19×

而是可能发现：

> 完整 Patch 1.20× → 删除负贡献/冲突编辑后 1.27×。

如果大部分任务只能做到“更小但不更快”，仍可作为 Pareto 权衡 结果，但论文主叙事要相应降低。

## 5. Motif Distiller（优化模式蒸馏器） 的处理

原报告把 归因模式复用 作为第二阶段贡献。修订后：

- **不作为硕士论文三个正式创新点之一**；
- 只作为 可选扩展；
- 必须与 SemOpt 的 strategy extraction/clustering/reuse 明确区分；
- 如果没有 留出 hit-rate/search-cost 改善，直接删除这一模块，不影响主框架成立。

## 6. Benchmark / 数据来源建议

优先级建议：

1. **SWE-Pro**：102 个真实专家优化任务，带参数化性能测试；适合真实仓库外验；
2. **SWE-Perf**：140 个真实 性能优化 PR 任务；适合 Patch/数据来源；
3. **SemOpt C/C++ tasks**：151 个优化任务，可用于生成更多候选 LLM Patch；
4. **自生成成功 Patch 池**：对 PolyBench/C、LLVM test-suite、性能热点使用多个 LLM/Agent 生成 Patch，只保留 `正确 + statistically significant 加速比 + 多编辑` 的样本。

注意：SWE-Pro 论文显示现有 LLM 可靠 运行时 gain 很少，因此不能假设直接拿到大量“成功 LLM Patch”；需要“真实专家 Patch + LLM 生成 Patch + 可控人工 交互 Patch”三类数据共同支撑。

## 7. 关键实验基线

- 原始完整 LLM Patch；
- ddmin / TRIM-style 最小化；
- LOO；
- 随机子集；
- approximate Shapley；
- sparse linear / LASSO 交互；
- GBDT 交互模型；
- 穷举 Oracle（仅小 Patch）；
- 纯静态编辑单元划分 vs LLM 辅助语义划分。

主要指标：

- final 运行时 / 加速比；
- relative 加速比 vs 完整 Patch；
- Patch 复杂度 / edit count；
- 交互恢复质量（在可控合成或 可穷举的小 Patch 上）；
- 测量预算；
- repeated-session stability；
- 正确性 pass rate；
- cross-input / 留出输入鲁棒性。

## 8. 当前查新结论

| 项目 | 2026-08-18 原版 | 2026-08-19 修订版 |
|---|---|---|
| 核心目标 | 归因 + 最小快速 Patch | **性能贡献/交互 + 更优重组** |
| Patch 最小化 | 核心模块/卖点 | **降级为结果和基线** |
| TRIM | 漏检 | **新增最强直接近邻** |
| Motif Distiller（优化模式蒸馏器） | 第二阶段贡献 | **降为 可选扩展** |
| 论文三创新 | 不够集中 | **1 主 + 2 支撑，边界清楚** |
| 最终性能故事 | 尽量保留原 加速比 | **争取达到/超过 完整 LLM Patch；否则报告 Pareto 权衡** |
| 当前新颖性判断 | partial | **partial，但联合边界更稳** |

## 9. CostWitness 状态

CostWitness 保留为条件性备选，但不建议与 PatchCredit 同时开发。其 后端信息感知 costing / legal-plan enumeration 与 LLVM VPlan 和已有 cost-model 工作碰撞较强。只有在先验小实验中证明“structured disagreement → 运行时 misranking”且 校准/GBDT/bandit 不足时，才值得恢复。

## 10. 诚信边界

- 当前没有运行任何新性能实验；本文所有数字均为 Benchmark 规模、文献结果或未来 门槛（Gate），不是 PatchCredit 实验结果。
- `llvm-mca`/静态 proxy 只能用于筛选，不能替代最终 运行时。
- 性能贡献 必须使用重复运行、固定环境和统计置信；不得从一次测量推出“因果”。
- 最终投稿前需再次执行增量查新，尤其检查 2026 年下半年新的 Agent Patch 归因/最小化 工作。
