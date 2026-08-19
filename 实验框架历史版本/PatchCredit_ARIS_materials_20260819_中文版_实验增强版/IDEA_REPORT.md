# IDEA REPORT：PatchCredit 修订版研究框架

> 日期：2026-08-19  
> ARIS 流程：`research-lit → idea-creator → novelty-check → competitor re-check → revision`  
> 状态：**已完成文献筛查；Pilot 尚未运行**  
> 主框架：**PatchCredit**  
> 备选：CostWitness（仅条件性备选）

## 1. 结论先行

PatchCredit 建议保留，但需要从 2026-08-18 原版进行结构性修订。

原版核心偏向：

> LLM 性能优化 Patch → 反事实删除 → 归因 → 最小快速 Patch。

在加入 2026-07 的 TRIM 后，“Agent Patch 的反事实删除/最小化”已经不能承担核心新颖性。修订版改成：

> **LLM 性能优化 Patch → 依赖闭合编辑单元 → 预算化运行时反事实测量 → 编辑级性能贡献 + 交互建模 → 性能引导的 Patch 重组 → 正确性 + 运行时验证。**

研究重点从“把 Patch 变小”升级为：

> **解释一个成功 LLM 多编辑优化 Patch 的真实性能来源，并利用编辑贡献和协同/冲突结构找到性能更好的组合。**

推荐当前评分：**8.3/10（研究框架层面，尚未经过 Pilot）**。这个评分不是论文录用概率，而是综合研究问题清晰度、新颖性边界、工程可行性、Benchmark 条件和硕士阶段完成风险后的内部评价。

---

## 2. 一句话定义

**PatchCredit：通过正确性门控的真实运行反事实实验，对 LLM 生成的多编辑优化 Patch 进行编辑级性能贡献与交互归因，并在有限测量预算下据此重组出正确且达到或超过原 Patch 性能的优化版本。**

---

## 3. 它到底解决什么问题

LLM 做性能优化时，经常一次修改多个位置。假设原程序运行 100 ms，LLM 一次进行了 A/B/C/D 四个编辑，完整 Patch 运行 70 ms。

只看完整 Patch，我们只知道：

> “LLM 成功加速了 30%。”

但不知道：

- A 是否真正有贡献；
- B/C 单独几乎无效、组合后是否出现协同；
- D 是否是负贡献；
- A 是否只是为了让 B 合法，而不是直接提升性能；
- 删除 D 或改变组合后，能否从 70 ms 进一步降到 68 ms。

PatchCredit 的核心任务就是把“整个 Patch 的一个 加速比 数字”拆成一个可验证的**性能贡献与交互结构**，再利用这个结构做新的优化决策。

---

## 4. 修订后的研究问题

### RQ1：编辑级性能贡献能否被稳定估计？

给定一个正确且显著加速的多编辑 Patch，构建 依赖闭合编辑单元，在重复 运行时 测量中估计各编辑的 边际性能贡献，并给出不确定性。

### RQ2：编辑间 非加性交互 是否真实且重要？

测试：

\[
I(A,B)=\Delta(A+B)-\Delta(A)-\Delta(B)
\]

其中正值可代表 协同，负值代表 冲突。真正有价值的案例是 `A`、`B` 单独收益小，但组合收益大，或者完整 Patch 因某个冲突编辑反而低于最佳子集。

### RQ3：能否用更少运行预算恢复足够有用的贡献/交互结构？

当编辑单元数量增加时，不可能总是跑完 `2^n` 个子集。需要预算化采样，优先测试对当前决策最有信息量的组合。

### RQ4：这些 贡献/交互 能否产生更好的 Patch？

最终不能只输出解释。必须把结构用于 重组，并实测：

- 是否比 完整 LLM Patch 更快；或
- 是否在几乎不损失性能的情况下显著降低 Patch 复杂度；
- 是否优于 LOO/ddmin/随机/非 LLM 交互 模型。

---

## 5. 三个正式创新点

### 5.1 主创新：依赖感知的性能贡献与交互建模（依赖感知的性能贡献与交互建模（Dependency-aware Performance Credit & Interaction） Modeling）

这是论文最核心的一点。

#### 输入

一个满足以下条件的 LLM 优化 Patch：

1. 多编辑；
2. 能编译；
3. 通过 正确性 tests；
4. 完整 Patch 相对 Baseline 有统计显著的 运行时 加速比。

#### 核心方法

Patch Unitizer 将 diff 转成依赖闭合编辑单元：

- AST 结构；
- def-use；
- 控制依赖；
- 声明/使用依赖；
- LLM 提出的 语义分组（需静态检查验证）。

随后对可行编辑子集执行：

> 编译 → 正确性测试 → 预热 → 重复运行时测量 → 置信估计。

输出：

- 编辑单元级边际贡献；
- 二阶 / 选择性高阶交互；
- 置信度 / 稳定性；
- 依赖图 + 性能交互图。

#### 为什么它是主创新

TRIM 关注功能上不必要的 Agent 编辑；PatchCredit 修订版关注的是**运行时性能 贡献**。

ECCO 已研究 Pass 级性能证据/协同；PatchCredit 必须证明在“生成式 Patch 内的异质编辑”这个对象上，依赖闭合 + 运行时反事实测量 + 交互 建模有不同的方法与价值。

---

### 5.2 支撑创新 1：预算化交互采样（Budgeted Interaction Sampling）

如果 `n=10`，理论上就有 1024 个编辑组合；每个组合还要多次 运行时 测量，直接穷举成本迅速失控。

因此设计一个预算化采样器，每轮根据：

- 依赖可行性；
- 当前 贡献估计不确定性；
- 交互残差；
- 正确性失败历史；
- 预期决策价值；
- 执行成本；

选择下一批最值得测的子集。

#### 目标

不是“设计一个名字新的采样算法”，而是证明：

> 在同样的最终 重组 质量 / 交互恢复质量 下，用更少真实运行次数完成分析。

#### 强基线

- 随机子集；
- LOO；
- ddmin；
- 近似 Shapley 采样；
- 稀疏线性模型/LASSO；
- GBDT 交互模型；
- 穷举 Oracle（仅限小 Patch）。

---

### 5.3 支撑创新 2：性能引导的 Patch 重组（Performance-guided Patch Recomposition）

原版的 `最小快速重组器（Minimal Fast Recomposer）` 目标太偏“精简”。修订后改为：

\[
\max_{S \subseteq P}
\quad
\mathrm{加速收益}(S)-\lambda\,\mathrm{复杂度}(S)
\]

subject to：

\[
\mathrm{正确性}(S)=1
\]

其中 `复杂度` 初期可以用 edit-编辑单元 数量或 diff size。

系统利用已估计的：

- 正贡献；
- 负贡献；
- 协同；
- 冲突；
- 依赖关系；
- 不确定性；

搜索候选组合，再对少量 Top 候选 做真实运行复核。

#### 最理想结果

Baseline：100 ms  
Full LLM Patch：70 ms  
PatchCredit Recomposition：**68 ms**

也就是删掉/替换负贡献或冲突编辑后，**比原始成功 Patch 还快**。

#### 次优但仍可发表的结果

Full Patch 70 ms、20 个编辑；Recomposed Patch 71 ms、10 个编辑。

这属于 加速比/复杂度 Pareto 改善，但论文叙事要比“更快”弱一些。

---

## 6. LLM 在框架里到底做什么

这是必须控制好的风险点。

**不能**让 LLM 根据文字主观判断“哪个编辑更快”，所有性能结论必须由执行证据产生。

LLM 的合理职责只有两类：

1. **语义编辑单元提议（Semantic Unit Proposal）**：根据 Patch 语义提出哪些 diff fragments 属于同一个优化意图/必须共同出现；
2. **交互假设提议（Interaction Hypothesis Proposal）**：根据数据流、控制流和优化语义提出“哪些 编辑单元 值得优先做组合测试”。

然后由 AST/def-use/compiler/test/运行时 验证。

必须做消融：

- 纯静态编辑单元划分器；
- LLM 辅助编辑单元划分器；
- 随机交互假设；
- regression/GBDT 交互模型 selection。

若 LLM 不能减少 测量预算 或提高重组性能，不应强行把“LLM 组件”写成创新；论文仍可定位为“对 LLM-generated optimization Patches 的 编译器性能分析”。

---

## 7. 与最强竞品的区别

### 7.1 TRIM

TRIM：

> Agent Patch → 找功能上不必要的 CodeSlop → 执行验证 → 更小 Patch。

PatchCredit：

> successful 性能 Patch → 测各编辑 运行时 贡献 + 协同/冲突 → 用性能结构重组 → 更快/更优 Pareto Patch。

**差别核心：功能必要性 vs 运行时性能贡献。**

### 7.2 ECCO

ECCO 的对象是 编译器 Pass 序列，研究 Pass 级性能证据 / 优化意图 / 序列搜索。

PatchCredit 的对象是一个自然生成的 源码/IR Patch 内的异质编辑单元，并对其做 正确性门控 运行时反事实测量。

### 7.3 Muppet

Muppet 已证明 保持正确性的 Mutation 子集 可以用于性能搜索。因此 PatchCredit 不能把“找更快 子集”本身当创新；必须显示：

> 依赖感知的性能贡献 + 交互估计 + 预算化测量 能带来更可靠或更低成本的 重组。

### 7.4 SWE-Pro

SWE-Pro 评价整项性能 Patch 的 运行时/memory，并处理输入和噪声变化；它不负责把 完整 Patch 加速比 分配到内部编辑，也不做基于内部 贡献 的重组。

### 7.5 SemOpt

SemOpt 已从历史优化修改中抽取、描述、聚类并复用 optimization strategies。因此原版 `Motif Distiller（优化模式蒸馏器）` 不再列为三个正式创新点，只保留 可选扩展。

---

## 8. 完整框架流程

```text
已成功加速的 LLM 优化 Patch
        │
        ▼
┌──────────────────────────┐
│ 1. Patch 编辑单元划分器  │
│ AST / def-use / 控制依赖 │
│ + LLM 语义编辑单元提议   │
└────────────┬─────────────┘
             │ 依赖闭合编辑单元
             ▼
┌──────────────────────────┐
│ 2. 正确性门控            │
│ 编译 + 测试              │
└────────────┬─────────────┘
             │ 仅保留合法子集
             ▼
┌──────────────────────────┐
│ 3. 预算化采样器          │
│ 不确定性 + 交互残差      │
│ + 依赖关系 + 测量成本    │
└────────────┬─────────────┘
             │ 选中的反事实子集
             ▼
┌──────────────────────────┐
│ 4. 真实运行时测量        │
│ 预热 + 重复执行 + CI     │
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 5. 性能贡献与交互建模    │
│ 边际贡献 / 协同 / 冲突   │
│ + 不确定性               │
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 6. Patch 重组器          │
│ 在正确性/复杂度约束下    │
│ 最大化实测性能收益       │
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 7. 最终验证              │
│ 测试 + 留出输入          │
│ + 重复运行时测量         │
└──────────────────────────┘
```

---

## 9. Benchmark 与数据设计

### 9.1 三类 Patch 数据

#### A. 真实性能优化 Patch

- SWE-Pro：102 个 expert optimizations；
- SWE-Perf：140 个真实 性能 PR 任务；
- 其他可复现 C/C++ 性能优化 PR。

用途：验证方法不是只在人工 toy case 上有效。

#### B. LLM 生成的成功 Patch

对现有 Benchmark 让多个模型/agent 生成优化 Patch，只保留：

- 正确性 pass；
- 多 edit 编辑单元；
- 完整 Patch 运行时 相对 Baseline 具有统计显著优势。

这是 PatchCredit 最核心的数据。

#### C. 可控 交互 Patch

人工/脚本构造少量已知 交互 structure 的 C/C++ kernels，例如：

- loop transformation + data-layout；
- strength reduction + hoisting；
- vectorization-enabling edit + alignment/alias edit；
- allocation removal + reuse；
- branch simplification + data reordering。

用途：验证 贡献/交互 recovery 是否真的正确，而不是只看最终 加速比。

### 9.2 建议 Benchmark

- SWE-Pro / SWE-Perf：真实仓库级外验；
- SemOpt C/C++ task set：候选优化任务池；
- PolyBench/C：稳定 kernel 与可控输入；
- LLVM test-suite 中适合 运行时 的子集；
- 可补 TSVC/LORE 作为编译优化 交互 MicroBenchmark，但不要只靠 MicroBenchmark 写最终结论。

---

## 10. 实验指标

### 10.1 性能结果

- absolute 运行时；
- 加速比 over Baseline；
- 加速比 over 完整 LLM Patch；
- worst-case regression；
- 留出输入鲁棒性。

### 10.2 归因结果

- 贡献排序稳定性；
- 二阶交互稳定性；
- synthetic/穷举 Oracle agreement；
- 不确定性 校准。

### 10.3 搜索成本

- 已测子集数；
- 程序总执行次数；
- 实际墙钟测量成本；
- 编译/测试失败造成的浪费。

### 10.4 Patch 复杂度

- 编辑单元数量；
- 修改行数；
- 依赖图 size；
- optional 可维护性代理指标（只做辅助，不做性能主张）。

---

## 11. 关键 Baselines

1. 完整 LLM Patch；
2. TRIM 风格层次化最小化；
3. ddmin；
4. LOO；
5. approximate Shapley；
6. 随机子集 sampling；
7. 稀疏线性模型/LASSO 交互；
8. GBDT 交互模型；
9. 穷举 Oracle（小 Patch）；
10. 纯静态 vs LLM 辅助编辑单元划分。

其中最关键的不是击败 随机，而是证明 **同预算下相对 LOO/ddmin/GBDT 的价值**。

---

## 12. 最小可行 Pilot

### Phase 0：数据 门槛（Gate）

先找到 **10–15 个**：

- 正确；
- 完整 Patch 有稳定 加速比；
- 4–8 个 依赖闭合单元；
- 至少存在多个可运行正确子集。

如果找不到，停止，不要先写复杂算法。

### Phase 1：小 Patch 穷举真值

对 4–8 编辑单元 Patch 尽可能穷举所有合法子集，建立：

- true 子集 性能 table；
- LOO；
- pairwise 交互；
- best 子集/重组。

先确认研究现象存在：

> 多编辑 Patch 中是否真的经常存在负贡献、协同、冲突，以及 完整 Patch 是否并非最优子集。

### Phase 2：Budgeted sampler

在 穷举真值 上离线模拟不同预算：25%、50%、75% 子集s，比较各方法找到 最佳/近最优重组 的成功率。

### Phase 3：真实大 Patch

扩到 8–15 编辑单元，不能穷举，使用 预算化采样器 + Top 候选验证。

---

## 13. Pilot 成功线与 终止条件（Kill Conditions）

### 建议成功线

不是论文结果，只是决定是否继续：

- ≥30% 的入选多编辑 Patch 存在显著 非加性交互；
- ≥20% 的 Patch 存在一个 保持正确性的子集/重组，其 运行时 显著优于 完整 Patch，**或**出现明确 加速比/复杂度 Pareto 改善；
- 预算化采样器 在 50% 以下 穷举 测量预算 时仍能以较高概率找到 近最优 重组；
- repeated-session 贡献 ranking 具有可接受稳定性；
- LLM 辅助的编辑单元划分/交互假设 至少在 测量预算 或 合法子集比例 上优于 纯静态方法。

### 终止条件（Kill Conditions）

出现以下情况应停止或大幅降级：

1. 多数 Patch 依赖闭合后只有 1–2 个不可分 编辑单元；
2. 通过正确性验证的子集 太少，交互 无法估计；
3. 性能噪声 大于 子集性能差异；
4. 几乎所有 完整 Patch 都已经是最优组合，没有可重组空间；
5. LOO/ddmin/GBDT 同预算 与 PatchCredit 持平；
6. LLM 语义单元和 交互 hypothesis 不带来任何收益。

---

## 14. 硕士论文“1 主 + 2 支撑”结构

| 层级 | 创新点 | 论文作用 |
|---|---|---|
| **主创新** | 依赖感知的性能贡献与交互建模（依赖感知的性能贡献与交互建模（Dependency-aware Performance Credit & Interaction） Modeling） | 回答“LLM 多编辑优化的性能到底从哪里来” |
| **支撑创新 1** | 预算化交互采样（Budgeted Interaction Sampling） | 解决真实 运行时反事实测量 实验太贵的问题 |
| **支撑创新 2** | 性能引导的 Patch 重组（Performance-guided Patch Recomposition） | 把分析结果重新用于性能优化，形成闭环 |

这三个点是一条因果式工程逻辑链，但论文表述上不需要声称严格 causal inference：

> **测得贡献结构 → 降低测量成本 → 用结构产生更优 Patch。**

---

## 15. 与 2026-08-18 原版的修改清单

| 原版 | 修订版 |
|---|---|
| 最小快速重组器（Minimal Fast Recomposer） 是核心卖点 | **性能引导重组器（Performance-guided Recomposer）**，最小化仅是复杂度正则 |
| 目标保留 ≥95% 完整 Patch 加速比、删除 ≥30% 编辑 | 目标优先 **达到/超过 完整 Patch 加速比**，95%/30% 只作为 Baseline 门槛（Gate） |
| 归因 比较宽泛 | 明确 **运行时 贡献 + 协同/冲突 + 不确定性** |
| 反事实采样器（Counterfactual Sampler） | 明确升级为 **预算化交互采样（Budgeted Interaction Sampling）** |
| Motif Distiller（优化模式蒸馏器） 第二阶段创新 | **降为 可选扩展** |
| 未覆盖 TRIM | **TRIM 加入最强直接近邻** |
| LLM 作用偏弱 | 收紧为 语义编辑单元提议 + 交互假设，并强制 消融实验 |

---

## 16. CostWitness（条件备选，不并行开发）

CostWitness 的基本想法仍是：当同一 LLVM pass 产生多个合法 plan，而 原生成本估计 与 代码生成后的资源证据 排序冲突时，触发 类型化 Witness + 稀疏真实运行时测量 进行纠偏。

但它仍有三个高风险：

1. 后端信息感知 costing 与 LLVM VPlan/现有 cost-model 工作直接相邻；
2. 强制 Plan 实现工程量较大；
3. 校准/GBDT/bandit 很可能已经足够。

因此只保留为备用，不建议与 PatchCredit 同时实现。

---

## 17. 最终研究叙事

### 不推荐的叙事

> LLM Patch 很冗余，所以我们把它删小，同时尽量保持性能。

这已经会被 TRIM/Muppet 等工作严重挤压。

### 推荐叙事

> **LLM 性能优化通常一次产生多个相互依赖的编辑，但整份 Patch 的 加速比 无法告诉我们哪些编辑真正贡献性能、哪些编辑发生协同或冲突。PatchCredit 通过依赖感知、正确性门控和噪声控制的反事实执行恢复编辑级性能贡献结构，并在有限测量预算下利用该结构重组出更优的正确优化 Patch。**

这就是修订版 PatchCredit 最值得保留的研究核心。

---

## 18. 关键参考

- TRIM: https://arxiv.org/abs/2607.18161
- ECCO: https://arxiv.org/abs/2602.00087
- SWE-Pro: https://arxiv.org/abs/2606.25530
- SWE-Perf: https://arxiv.org/abs/2507.12415
- Performance Benchmark reliability audit: https://arxiv.org/abs/2607.01211
- SemOpt: https://arxiv.org/abs/2510.16384
- Agents that Matter: https://arxiv.org/abs/2605.27621
- SBLLM: https://arxiv.org/abs/2408.12159
- Muppet: https://doi.org/10.1016/j.parco.2024.103097

> 说明：本报告仅完成研究设计与查新边界修订，没有运行 PatchCredit 实验；任何成功线均为可证伪 门槛（Gate），不是已有结果。
