# PatchCredit 修订版查新请求

> 重新审计日期：2026-08-19  
> 目的：在弱化“Patch 最小化”、强化“运行时性能贡献 + 编辑交互建模”之后，重新检查 PatchCredit 的创新边界。

## PatchCredit 修订版主张

- **A1 — 主创新主张：** 针对一个由 LLM 自然生成、通过正确性验证且能够产生加速的多编辑优化 Patch，在**依赖闭合的编辑单元**和**噪声受控的真实执行**条件下，估计**编辑级运行时性能贡献（Performance Credit）**以及**非加性交互（协同/冲突）**。
- **A2 — 支撑创新 1：** 利用依赖信息、当前估计不确定性和已经观测到的交互结构，执行**预算化反事实子集选择（Budgeted Counterfactual Subset Selection）**，相较于穷举全部组合，使用更少的编译/测试/运行实验，同时保留足够可靠的贡献与交互估计。
- **A3 — 支撑创新 2：** 利用实测得到的贡献和交互信息进行**性能引导的 Patch 重组（Performance-guided Recomposition）**，目标是在保持正确性的前提下寻找更优编辑组合；目标并不仅是得到更小的 Patch，还允许通过删除负贡献或冲突编辑，使重组后的性能超过原始完整 LLM Patch。
- **A4 — LLM 的职责：** LLM 仅用于提出语义编辑单元和交互假设；所有合法性、正确性和性能结论均必须由编译器/静态检查、测试以及真实运行时测量验证。

## 明确删除或降级的主张

- “反事实删除 / Patch 最小化”**不再作为独立创新点**。
- “删除 30% 编辑同时保留 95% 加速”只是评价门槛，而不是创新点。
- “Motif Distiller / 优化模式复用”仅作为可选扩展，不作为硕士论文正式创新点。
- 除非能够严格说明可识别性假设，否则不使用严格的“因果贡献”表述；默认术语为 `performance credit / counterfactual contribution estimate`（性能贡献 / 反事实贡献估计）。

## 必须比较的工作和基线

TRIM（2026）、ECCO（2026）、Muppet（2024）、SWE-Pro（2026）、SWE-Perf/GSO/SWE-efficiency 可靠性审计（2026）、Agents that Matter（2026）、SemOpt（2025/2026）、SBLLM，以及 Delta Debugging、LOO、Shapley 近似、稀疏回归/GBDT 交互模型。

## CostWitness

仅保留为条件性备选。沿用上一轮审计结论：backend-informed costing（后端信息感知成本建模）已有较强先验工作；只有“结构化分歧触发 + 类型化 witness + 稀疏真实测量”还可能存在空间，而且必须先证明简单 calibration/GBDT/bandit 基线不足。
