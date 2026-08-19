# 独立创新性审计结果——PatchCredit 修订版

> 截至 2026-08-19。判定词：`direct`（直接覆盖）、`partial`（部分重叠）、`novel/not found`（未发现直接覆盖）。  
> 总体结论：**PatchCredit 修订版 = 存在部分先验重叠，但组合创新边界仍可辩护。** 相比 2026-08-18 版本，新版明显更安全，因为 Patch 最小化已经不再被当作核心贡献。

## 1. 新增强竞品后的关键变化

本轮新增的最强碰撞工作是 **TRIM: Reducing AI-Generated CodeSlop via Agent Trajectory Minimization**（2026-07-20，arXiv:2607.18161）。TRIM 已经对 Agent 生成编辑执行层次化反事实删除，通过真实执行验证删除是否可行，并以比 Delta Debugging 更高效的方式获得更小 Patch。因此：

- “Agent Patch → 删除编辑 → 执行验证 → 得到更小 Patch”不能再作为 PatchCredit 的创新；
- 仅仅做 dependency-aware / hierarchical Patch minimization 也不够新；
- PatchCredit 必须把核心收紧到**运行时性能贡献（Performance Credit）和编辑交互结构（Interaction Structure）**。

TRIM：https://arxiv.org/abs/2607.18161

## 2. 修订后的逐项创新判定

| 主张 | 判定 | 最接近先验工作 | 可守创新边界 / Kill Condition |
|---|---|---|---|
| A1：LLM 性能 Patch 内的编辑级运行时贡献 + 交互 | **部分重叠；未发现完全相同的组合边界** | ECCO；Muppet；Agents that Matter；TRIM | 只有当系统真正对依赖闭合编辑进行真实性能贡献测量，并在运行噪声下显式建模协同/冲突时才可守。若同预算下 LOO/简单回归即可给出同等稳定决策，则该点被削弱。 |
| A2：预算化反事实子集选择 | **部分重叠** | Delta Debugging；主动子集选择；归因文献 | 必须证明在相同重组质量/交互恢复质量下显著减少真实执行次数，不能单独过度声称采样算法新颖。 |
| A3：性能引导的 Patch 重组 | **部分重叠；比“最小化”叙事更强** | Muppet；TRIM；通用搜索 | 价值来自利用实测 credit/interaction 找到更好的正确编辑组合；“Patch 更小”本身不够。若最终只是复现 ddmin/LOO 的结果，则该点失去意义。 |
| A4：LLM 语义单元划分 / 交互假设 | **部分重叠且风险较高** | SemOpt；Compiler Agent 系统 | 必须与纯 AST/def-use 分组、非 LLM 交互模型做消融。若 LLM 不能减少搜索成本或提升泛化/重组效果，应把论文定位成“LLM 生成 Patch 的性能分析”，而不是“LLM 方法”。 |

## 3. 重要近邻工作

- **ECCO** 研究编译器 Pass 序列优化中的证据驱动推理、性能影响和协同关系，因此“性能 evidence / synergy”作为一般概念并不新；但它的对象是 Pass 序列，而不是生成式 Patch 内部的异质编辑。  
  https://arxiv.org/abs/2602.00087
- **Muppet** 已经在保持正确性的前提下搜索 mutation 子集以获得更好性能，所以“为了性能做编辑子集裁剪/搜索”本身不是新贡献。  
  https://doi.org/10.1016/j.parco.2024.103097
- **SWE-Pro** 对仓库级性能 Patch 进行参数化、噪声感知的运行时间/内存评测，因此多输入、噪声感知的 Patch 评测不能声称为创新。  
  https://arxiv.org/abs/2606.25530
- **性能 Benchmark 可靠性审计** 表明跨机器和运行环境的不稳定性可能主导性能结论，因此重复测量和置信区间在 PatchCredit 中属于必要条件。  
  https://arxiv.org/abs/2607.01211
- **SemOpt** 已经从历史优化修改中提取、描述、聚类并复用优化策略，因此 `Motif Distiller` 应保留为可选扩展，而不应作为论文正式贡献。  
  https://arxiv.org/abs/2510.16384

## 4. 推荐的硕士论文创新结构

1. **主创新——依赖感知的性能贡献与交互建模（Dependency-aware Performance Credit & Interaction Modeling）**  
   通过正确性门控、噪声控制的反事实真实执行，估计每个编辑单元的运行时贡献以及二阶/高阶协同与冲突。
2. **支撑创新 1——预算化交互采样（Budgeted Interaction Sampling）**  
   不穷举全部 `2^n` 组合，而是优先选择最有信息量的合法子集进行实测。
3. **支撑创新 2——性能引导的 Patch 重组（Performance-guided Patch Recomposition）**  
   在正确性约束下搜索能够最大化实测加速的编辑组合；结果可以比原始完整 Patch 更小，也可以更快。

## 5. 推荐表述

安全的一句话表述：

> PatchCredit 对已经成功加速的 LLM 多编辑优化 Patch 进行依赖感知的编辑级运行时贡献与交互测量，并利用这些实测结构，在有限测量预算下重组出保持正确且具有相同或更好运行性能的 Patch。

应避免的表述：

> 我们首次利用反事实删除最小化 LLM Patch。

## 6. Pilot 进入条件

只有在小规模 Patch 上同时验证以下条件后，才值得继续投入：

- 每个入选 Patch 至少存在 4–8 个有意义、依赖闭合的编辑单元；
- 多轮 runtime session 中，完整 Patch 加速和关键子集比较足够稳定；
- 非加性交互出现频率足够高，确实会影响决策；
- 在显著减少真实运行次数的情况下，预算化采样能够达到或接近 exhaustive/LOO 的重组质量；
- 在有意义比例的任务上，重组结果能够达到或超过完整 Patch 的性能，或取得明确的“加速-复杂度”Pareto 改善；
- LLM 辅助的分组/交互假设相对于纯静态分析确实能带来收益。

## 7. CostWitness

继续保持**条件性备选**。此前风险不变：LLVM VPlan 和已有 backend-informed cost modeling 已覆盖其大量核心空间。除非小规模数据审计先证明存在稳定的 runtime misranking（运行时排序错误），且简单 calibration/GBDT/bandit 无法解决，否则不建议和 PatchCredit 并行开发。
