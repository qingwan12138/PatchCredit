# IDEA REPORT：PatchCredit 修订版研究框架

> 日期：2026-08-19  
> ARIS 流程：`research-lit → idea-creator → novelty-check → competitor re-check → revision`  
> 状态：**Literature-screened；pilot 尚未运行**  
> 主框架：**PatchCredit**  
> 备选：CostWitness（conditional only）

## 1. 结论先行

PatchCredit 建议保留，但需要从 2026-08-18 原版进行结构性修订。

原版核心偏向：

> LLM performance patch → 反事实删除 → 归因 → 最小快速 patch。

在加入 2026-07 的 TRIM 后，“agent patch 的反事实删除/最小化”已经不能承担核心新颖性。修订版改成：

> **LLM performance patch → dependency-closed edit units → budgeted runtime counterfactual measurements → edit-level performance credit + interaction modeling → performance-guided recomposition → correctness + runtime validation。**

研究重点从“把 patch 变小”升级为：

> **解释一个成功 LLM 多编辑优化 patch 的真实性能来源，并利用编辑贡献和协同/冲突结构找到性能更好的组合。**

推荐当前评分：**8.3/10（研究框架层面，尚未经过 pilot）**。这个评分不是论文录用概率，而是综合研究问题清晰度、新颖性边界、工程可行性、benchmark 条件和硕士阶段完成风险后的内部评价。

---

## 2. 一句话定义

**PatchCredit：通过正确性门控的真实运行反事实实验，对 LLM 生成的多编辑优化 Patch 进行编辑级性能贡献与交互归因，并在有限测量预算下据此重组出正确且达到或超过原 Patch 性能的优化版本。**

---

## 3. 它到底解决什么问题

LLM 做性能优化时，经常一次修改多个位置。假设原程序运行 100 ms，LLM 一次进行了 A/B/C/D 四个编辑，完整 patch 运行 70 ms。

只看完整 patch，我们只知道：

> “LLM 成功加速了 30%。”

但不知道：

- A 是否真正有贡献；
- B/C 单独几乎无效、组合后是否出现协同；
- D 是否是负贡献；
- A 是否只是为了让 B 合法，而不是直接提升性能；
- 删除 D 或改变组合后，能否从 70 ms 进一步降到 68 ms。

PatchCredit 的核心任务就是把“整个 patch 的一个 speedup 数字”拆成一个可验证的**性能贡献与交互结构**，再利用这个结构做新的优化决策。

---

## 4. 修订后的研究问题

### RQ1：编辑级性能贡献能否被稳定估计？

给定一个正确且显著加速的多编辑 patch，构建 dependency-closed edit units，在重复 runtime 测量中估计各编辑的 marginal performance credit，并给出不确定性。

### RQ2：编辑间 non-additive interaction 是否真实且重要？

测试：

\[
I(A,B)=\Delta(A+B)-\Delta(A)-\Delta(B)
\]

其中正值可代表 synergy，负值代表 conflict。真正有价值的案例是 `A`、`B` 单独收益小，但组合收益大，或者完整 patch 因某个冲突编辑反而低于最佳子集。

### RQ3：能否用更少运行预算恢复足够有用的贡献/交互结构？

当编辑单元数量增加时，不可能总是跑完 `2^n` 个子集。需要预算化采样，优先测试对当前决策最有信息量的组合。

### RQ4：这些 credit/interaction 能否产生更好的 patch？

最终不能只输出解释。必须把结构用于 recomposition，并实测：

- 是否比 full LLM patch 更快；或
- 是否在几乎不损失性能的情况下显著降低 patch complexity；
- 是否优于 LOO/ddmin/随机/非 LLM interaction 模型。

---

## 5. 三个正式创新点

### 5.1 主创新：Dependency-aware Performance Credit & Interaction Modeling

这是论文最核心的一点。

#### 输入

一个满足以下条件的 LLM optimization patch：

1. 多编辑；
2. 能编译；
3. 通过 correctness tests；
4. full patch 相对 baseline 有统计显著的 runtime speedup。

#### 核心方法

Patch Unitizer 将 diff 转成依赖闭合编辑单元：

- AST 结构；
- def-use；
- control dependency；
- declaration/use dependency；
- LLM 提出的 semantic grouping（需静态检查验证）。

随后对可行编辑子集执行：

> build → correctness test → warmup → repeated runtime measurement → confidence estimate。

输出：

- unit-level marginal credit；
- pairwise / selected higher-order interaction；
- confidence / stability；
- dependency graph + performance interaction graph。

#### 为什么它是主创新

TRIM 关注功能上不必要的 agent edits；PatchCredit 修订版关注的是**运行时性能 credit**。

ECCO 已研究 pass-level performance evidence/synergy；PatchCredit 必须证明在“heterogeneous edits inside a generated patch”这个对象上，dependency closure + runtime counterfactual + interaction 建模有不同的方法与价值。

---

### 5.2 支撑创新 1：Budgeted Interaction Sampling

如果 `n=10`，理论上就有 1024 个编辑组合；每个组合还要多次 runtime 测量，直接穷举成本迅速失控。

因此设计一个预算化采样器，每轮根据：

- dependency feasibility；
- 当前 credit uncertainty；
- interaction residual；
- correctness failure history；
- expected decision value；
- execution cost；

选择下一批最值得测的子集。

#### 目标

不是“设计一个名字新的采样算法”，而是证明：

> 在同样的最终 recomposition 质量 / interaction recovery 下，用更少真实运行次数完成分析。

#### 强基线

- random subset；
- LOO；
- ddmin；
- approximate Shapley sampling；
- sparse linear/LASSO；
- GBDT interaction；
- exhaustive oracle（仅限小 patch）。

---

### 5.3 支撑创新 2：Performance-guided Patch Recomposition

原版的 `Minimal Fast Recomposer` 目标太偏“精简”。修订后改为：

\[
\max_{S \subseteq P}
\quad
\mathrm{Speedup}(S)-\lambda\,\mathrm{Complexity}(S)
\]

subject to：

\[
\mathrm{Correct}(S)=1
\]

其中 `Complexity` 初期可以用 edit-unit 数量或 diff size。

系统利用已估计的：

- positive credit；
- negative credit；
- synergy；
- conflict；
- dependency；
- uncertainty；

搜索候选组合，再对少量 top candidates 做真实运行复核。

#### 最理想结果

Baseline：100 ms  
Full LLM Patch：70 ms  
PatchCredit Recomposition：**68 ms**

也就是删掉/替换负贡献或冲突编辑后，**比原始成功 patch 还快**。

#### 次优但仍可发表的结果

Full Patch 70 ms、20 个编辑；Recomposed Patch 71 ms、10 个编辑。

这属于 speedup/complexity Pareto 改善，但论文叙事要比“更快”弱一些。

---

## 6. LLM 在框架里到底做什么

这是必须控制好的风险点。

**不能**让 LLM 根据文字主观判断“哪个编辑更快”，所有性能结论必须由执行证据产生。

LLM 的合理职责只有两类：

1. **Semantic Unit Proposal**：根据 patch 语义提出哪些 diff fragments 属于同一个优化意图/必须共同出现；
2. **Interaction Hypothesis Proposal**：根据数据流、控制流和优化语义提出“哪些 unit 值得优先做组合测试”。

然后由 AST/def-use/compiler/test/runtime 验证。

必须做消融：

- static-only unitizer；
- LLM-aided unitizer；
- random interaction proposal；
- regression/GBDT interaction selection。

若 LLM 不能减少 measurement budget 或提高重组性能，不应强行把“LLM 组件”写成创新；论文仍可定位为“对 LLM-generated optimization patches 的 compiler performance analysis”。

---

## 7. 与最强竞品的区别

### 7.1 TRIM

TRIM：

> agent patch → 找功能上不必要的 CodeSlop → 执行验证 → 更小 patch。

PatchCredit：

> successful performance patch → 测各编辑 runtime credit + synergy/conflict → 用性能结构重组 → 更快/更优 Pareto patch。

**差别核心：functional necessity vs runtime performance contribution。**

### 7.2 ECCO

ECCO 的对象是 compiler pass sequence，研究 pass-level evidence/optimization intent/sequence search。

PatchCredit 的对象是一个自然生成的 source/IR patch 内异质 edit units，并对其做 correctness-gated runtime counterfactual measurement。

### 7.3 Muppet

Muppet 已证明 correctness-preserving mutation subsets 可以用于性能搜索。因此 PatchCredit 不能把“找更快 subset”本身当创新；必须显示：

> dependency-aware credit + interaction estimation + budgeted measurement 能带来更可靠或更低成本的 recomposition。

### 7.4 SWE-Pro

SWE-Pro 评价整项性能 patch 的 runtime/memory，并处理输入和噪声变化；它不负责把 full-patch speedup 分配到内部编辑，也不做基于内部 credit 的重组。

### 7.5 SemOpt

SemOpt 已从历史优化修改中抽取、描述、聚类并复用 optimization strategies。因此原版 `Motif Distiller` 不再列为三个正式创新点，只保留 optional extension。

---

## 8. 完整框架流程

```text
Successful LLM Optimization Patch
        │
        ▼
┌──────────────────────────┐
│ 1. Patch Unitizer        │
│ AST / def-use / control  │
│ + LLM semantic proposal  │
└────────────┬─────────────┘
             │ dependency-closed units
             ▼
┌──────────────────────────┐
│ 2. Correctness Gate      │
│ build + tests            │
└────────────┬─────────────┘
             │ valid subsets only
             ▼
┌──────────────────────────┐
│ 3. Budgeted Sampler      │
│ uncertainty + interaction│
│ + dependency + cost      │
└────────────┬─────────────┘
             │ selected counterfactual subsets
             ▼
┌──────────────────────────┐
│ 4. Runtime Measurement   │
│ warmup + repeats + CI    │
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 5. Credit & Interaction  │
│ marginal / synergy /     │
│ conflict / uncertainty   │
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 6. Patch Recomposer      │
│ maximize speedup under   │
│ correctness/complexity   │
└────────────┬─────────────┘
             ▼
┌──────────────────────────┐
│ 7. Final Validation      │
│ tests + held-out inputs  │
│ repeated runtime         │
└──────────────────────────┘
```

---

## 9. Benchmark 与数据设计

### 9.1 三类 Patch 数据

#### A. 真实性能优化 Patch

- SWE-Pro：102 个 expert optimizations；
- SWE-Perf：140 个真实 performance PR tasks；
- 其他可复现 C/C++ performance PR。

用途：验证方法不是只在人工 toy case 上有效。

#### B. LLM 生成的成功 Patch

对现有 benchmark 让多个模型/agent 生成优化 patch，只保留：

- correctness pass；
- 多 edit units；
- full patch runtime statistically better than baseline。

这是 PatchCredit 最核心的数据。

#### C. 可控 interaction Patch

人工/脚本构造少量已知 interaction structure 的 C/C++ kernels，例如：

- loop transformation + data-layout；
- strength reduction + hoisting；
- vectorization-enabling edit + alignment/alias edit；
- allocation removal + reuse；
- branch simplification + data reordering。

用途：验证 credit/interaction recovery 是否真的正确，而不是只看最终 speedup。

### 9.2 建议 benchmark

- SWE-Pro / SWE-Perf：真实仓库级外验；
- SemOpt C/C++ task set：候选优化任务池；
- PolyBench/C：稳定 kernel 与可控输入；
- LLVM test-suite 中适合 runtime 的子集；
- 可补 TSVC/LORE 作为编译优化 interaction microbenchmark，但不要只靠 microbenchmark 写最终结论。

---

## 10. 实验指标

### 10.1 性能结果

- absolute runtime；
- speedup over baseline；
- speedup over full LLM patch；
- worst-case regression；
- held-out input robustness。

### 10.2 归因结果

- credit ranking stability；
- pairwise interaction stability；
- synthetic/exhaustive oracle agreement；
- uncertainty calibration。

### 10.3 搜索成本

- measured subsets；
- total program executions；
- wall-clock measurement cost；
- compile/test failure waste。

### 10.4 Patch 复杂度

- edit-unit count；
- lines changed；
- dependency graph size；
- optional maintainability proxy（只做辅助，不做性能主张）。

---

## 11. 关键 Baselines

1. full LLM patch；
2. TRIM-style hierarchical minimization；
3. ddmin；
4. LOO；
5. approximate Shapley；
6. random subset sampling；
7. sparse linear/LASSO interaction；
8. GBDT interaction；
9. exhaustive oracle（小 patch）；
10. static-only vs LLM-aided unitization。

其中最关键的不是击败 random，而是证明 **同预算下相对 LOO/ddmin/GBDT 的价值**。

---

## 12. 最小可行 Pilot

### Phase 0：数据 gate

先找到 **10–15 个**：

- correct；
- full patch 有稳定 speedup；
- 4–8 个 dependency-closed units；
- 至少存在多个可运行正确子集。

如果找不到，停止，不要先写复杂算法。

### Phase 1：小 patch exhaustive truth

对 4–8 unit patch 尽可能穷举所有合法子集，建立：

- true subset performance table；
- LOO；
- pairwise interaction；
- best subset/recomposition。

先确认研究现象存在：

> 多编辑 patch 中是否真的经常存在负贡献、synergy、conflict，以及 full patch 是否并非最优子集。

### Phase 2：Budgeted sampler

在 exhaustive truth 上离线模拟不同预算：25%、50%、75% subsets，比较各方法找到 best/near-best recomposition 的成功率。

### Phase 3：真实大 patch

扩到 8–15 units，不能穷举，使用 budgeted sampler + top-candidate validation。

---

## 13. Pilot 成功线与 Kill Conditions

### 建议成功线

不是论文结果，只是决定是否继续：

- ≥30% 的入选多编辑 patch 存在显著 non-additive interaction；
- ≥20% 的 patch 存在一个 correctness-preserving subset/recomposition，其 runtime 显著优于 full patch，**或**出现明确 speedup/complexity Pareto 改善；
- budgeted sampler 在 50% 以下 exhaustive measurement budget 时仍能以较高概率找到 near-best recomposition；
- repeated-session credit ranking 具有可接受稳定性；
- LLM-aided unitization/hypothesis 至少在 measurement budget 或 valid-subset rate 上优于 static-only。

### Kill Conditions

出现以下情况应停止或大幅降级：

1. 多数 patch 依赖闭合后只有 1–2 个不可分 unit；
2. correctness-passing subsets 太少，interaction 无法估计；
3. performance noise 大于 subset effect；
4. 几乎所有 full patch 都已经是最优组合，没有可重组空间；
5. LOO/ddmin/GBDT equal-budget 与 PatchCredit 持平；
6. LLM 语义单元和 interaction hypothesis 不带来任何收益。

---

## 14. 硕士论文“1 主 + 2 支撑”结构

| 层级 | 创新点 | 论文作用 |
|---|---|---|
| **主创新** | Dependency-aware Performance Credit & Interaction Modeling | 回答“LLM 多编辑优化的性能到底从哪里来” |
| **支撑创新 1** | Budgeted Interaction Sampling | 解决真实 runtime counterfactual 实验太贵的问题 |
| **支撑创新 2** | Performance-guided Patch Recomposition | 把分析结果重新用于性能优化，形成闭环 |

这三个点是一条因果式工程逻辑链，但论文表述上不需要声称严格 causal inference：

> **测得贡献结构 → 降低测量成本 → 用结构产生更优 patch。**

---

## 15. 与 2026-08-18 原版的修改清单

| 原版 | 修订版 |
|---|---|
| Minimal Fast Recomposer 是核心卖点 | **Performance-guided Recomposer**，最小化仅是复杂度正则 |
| 目标保留 ≥95% full-patch speedup、删除 ≥30% edits | 目标优先 **达到/超过 full-patch speedup**，95%/30% 只作为 baseline gate |
| attribution 比较宽泛 | 明确 **runtime credit + synergy/conflict + uncertainty** |
| Counterfactual Sampler | 明确升级为 **Budgeted Interaction Sampling** |
| Motif Distiller 第二阶段创新 | **降为 optional extension** |
| 未覆盖 TRIM | **TRIM 加入最强直接近邻** |
| LLM 作用偏弱 | 收紧为 semantic unit proposal + interaction hypothesis，并强制 ablation |

---

## 16. CostWitness（条件备选，不并行开发）

CostWitness 的基本想法仍是：当同一 LLVM pass 产生多个合法 plan，而 native cost 与 post-codegen resource evidence 排序冲突时，触发 typed witness + sparse runtime measurement 进行纠偏。

但它仍有三个高风险：

1. backend-informed costing 与 LLVM VPlan/现有 cost-model 工作直接相邻；
2. forced-plan 实现工程量较大；
3. calibration/GBDT/bandit 很可能已经足够。

因此只保留为备用，不建议与 PatchCredit 同时实现。

---

## 17. 最终研究叙事

### 不推荐的叙事

> LLM patch 很冗余，所以我们把它删小，同时尽量保持性能。

这已经会被 TRIM/Muppet 等工作严重挤压。

### 推荐叙事

> **LLM 性能优化通常一次产生多个相互依赖的编辑，但整份 patch 的 speedup 无法告诉我们哪些编辑真正贡献性能、哪些编辑发生协同或冲突。PatchCredit 通过依赖感知、正确性门控和噪声控制的反事实执行恢复编辑级性能贡献结构，并在有限测量预算下利用该结构重组出更优的正确优化 patch。**

这就是修订版 PatchCredit 最值得保留的研究核心。

---

## 18. 关键参考

- TRIM: https://arxiv.org/abs/2607.18161
- ECCO: https://arxiv.org/abs/2602.00087
- SWE-Pro: https://arxiv.org/abs/2606.25530
- SWE-Perf: https://arxiv.org/abs/2507.12415
- Performance benchmark reliability audit: https://arxiv.org/abs/2607.01211
- SemOpt: https://arxiv.org/abs/2510.16384
- Agents that Matter: https://arxiv.org/abs/2605.27621
- SBLLM: https://arxiv.org/abs/2408.12159
- Muppet: https://doi.org/10.1016/j.parco.2024.103097

> 说明：本报告仅完成研究设计与查新边界修订，没有运行 PatchCredit 实验；任何成功线均为可证伪 gate，不是已有结果。
