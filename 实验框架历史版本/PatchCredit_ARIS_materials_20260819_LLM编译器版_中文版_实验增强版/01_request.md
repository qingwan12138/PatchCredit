# 01_request：PatchCredit（LLM 编译器版）创新性复查请求

## 目标

对以下新版 PatchCredit 主张进行强竞品复核，重点判断是否已经被 2024–2026 年 LLM compiler auto-tuning、phase ordering、pass synergy、runtime evidence、per-pass attribution 工作覆盖。

## A1：主创新

给定 LLM Compiler Agent 已生成的多步骤优化计划：

\[
S=[a_1,\ldots,a_n]
\]

基于真实编译、正确性与 runtime 干预，估计：

\[
Credit(a_i\mid S)
\]

以及：

\[
Interaction(a_i,a_j\mid S)
\]

要求 action instance 与位置/参数/上下文绑定，不退化为全局 Pass 类型的重要性。

## A2：支撑创新 1

在不能穷举 \(2^n\) 个计划组合时，使用预算感知的反事实采样，优先选择：

- 高不确定性 action；
- 疑似强 interaction；
- 对当前重组决策影响最大的 intervention；
- 高可行性、低无效构建风险计划。

目标是在固定 execution budget 下恢复足以支持重组的 credit/interaction。

## A3：支撑创新 2

根据 credit 与 interaction 对原 LLM plan 做：

- order-preserving pruning；
- block pruning；
- synergy group 保留；
- conflict action 删除；
- 正式版可扩展局部 reorder。

最终必须通过真实 runtime 比较：

\[
S_{PC}
\quad vs \quad
S_{LLM}
\]

## A4：明确不主张的内容

不主张：

- 首次 LLM pass sequence tuning；
- 首次 pass synergy；
- 首次 causal/evidence compiler optimization；
- 首次 individual pass runtime contribution；
- 首次 runtime feedback LLM compiler agent；
- 首次删除/最小化 agent patch。

## 需要重点对比

- Compiler-R1
- ECCO
- Synergy-Guided Compiler Auto-Tuning
- AutoPass
- A Multi-Dimensional, Per-Pass Empirical Study of the LLVM Optimization Pipeline
- TRIM
- PassNet
- 经典 phase-ordering / LOO / Shapley / ddmin

## 审计标准

如果已有工作同时满足：

1. LLM-generated compiler plan；
2. post-hoc contextual action-instance credit；
3. real runtime counterfactual execution；
4. budgeted intervention selection；
5. credit-guided recomposition；

则 PatchCredit 应判为严重撞车。

若只有其中若干组件重叠，则给出必须缩窄的创新边界。
