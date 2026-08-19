# IDEA REPORT：PatchCredit（LLM 编译器版）

> 日期：2026-08-19  
> 状态：**创新边界已重新收紧；尚未运行 Pilot**  
> 研究方向：**LLM + Compiler Auto-Tuning / Compiler Optimization**  
> 主框架：**PatchCredit**  
> 当前版本定位：**LLM Compiler Agent 优化计划的上下文反事实性能归因与二次重组**

---

# 1. 结论先行

PatchCredit 当前不再研究“LLM 修改源代码以后，哪些 diff 有用”，而是明确收紧为：

> **LLM Compiler Agent 先生成一个多步骤编译优化计划，PatchCredit 再通过真实编译与运行，对计划中每个优化动作做上下文条件化的反事实性能归因，识别协同、冲突和冗余动作，并在有限测量预算下重组出更好的优化计划。**

因此它现在是一条明确的 **LLM Compiler** 主线，而不是泛化的软件性能工程框架。

一句话流程：

```text
Benchmark Program
      ↓
LLM Compiler Agent
      ↓
S_LLM = [a1, a2, ..., an]
      ↓
真实编译 + 正确性 + Runtime
      ↓
PatchCredit
  ├─ Contextual Credit
  ├─ Interaction
  ├─ Budgeted Counterfactual Sampling
  └─ Credit-guided Recomposition
      ↓
S_PC
      ↓
真实编译 + 正确性 + Runtime
```

最终比较：


```text
Original / Standard Compiler  vs  S_LLM  vs  S_PC
```


---

# 2. PatchCredit 到底解决什么问题

假设 LLM Compiler Agent 输出一个 LLVM 优化计划：


```text
S_LLM = [A, B, C, D, E, F]
```


它把某个程序从 100 ms 优化到 80 ms。

普通 LLM 编译器工作通常只关心：

> “这个 sequence 有没有得到更好的目标值？”

PatchCredit 继续追问：

- A 在当前 sequence 中到底贡献多少 runtime speedup？
- D 是否在当前上下文中实际造成回退？
- B、C 单独收益弱，但是否组合后出现显著协同？
- E 是否冗余？
- F 是否只有在 A 存在时才有效？
- 能否通过少量真实运行实验，避免穷举所有 \(2^n\) 个组合？
- 能否据此得到一个比完整 LLM sequence 更快的新计划？

因此 PatchCredit 的核心不是“哪个 Pass 平均最好”，而是：

> **某个具体 LLM-generated optimization plan 中，每个 action instance 在当前上下文里的真实性能贡献。**

---

# 3. 研究对象：Action Instance，而不是简单 Pass 类型

正式对象定义为：


```text
a_i = (pass_id, params, position, nesting/context)
```


例如同一个 `loop-unroll` 在不同位置出现两次，应视为两个 action instances，而不是一个“Pass 类型”。

原因：

1. Pass 的效果依赖前序 IR 状态；
2. 同一 Pass 在不同位置作用可能完全不同；
3. LLVM New Pass Manager 可能存在层次化 pipeline；
4. 参数不同也会改变作用；
5. PatchCredit 研究的是“这个动作在这个计划里”的贡献。

---

# 4. 当前三个创新点

## 主创新：Contextual Counterfactual Credit Assignment

对一个 LLM 生成的优化计划 \(S\)，估计：


```text
Credit(a_i | S)
```


不是只算简单 prefix 差值，也不是只问某个 Pass 的全局平均效果。

基础反事实可以从删除干预开始：


```text
Delta_i(S) = U(S) - U(S_without_a_i)
```


但正式版本不应停留在单次 LOO，而应在计划附近的多个**可行上下文**中估计：


```text
C_i = E_{C ~ q(. | S)} [ U(C + a_i) - U(C) ]
```


其中：

- \(U\) 由真实 runtime 定义；
- 编译失败/正确性失败的计划不能作为正常性能样本；
- 上下文采样受 compiler pipeline 约束；
- 输出应带置信区间或不确定性。

### Interaction

对两个动作：


```text
I_ij = U(C+i+j) - U(C+i) - U(C+j) + U(C)
```


用于识别：

- synergy：组合收益大于简单相加；
- conflict：组合后相互抵消或回退；
- conditional dependency：某动作只在特定上下文有效。

---

## 支撑创新 1：Budgeted Counterfactual Credit Estimation

当计划包含 \(n\) 个动作时，全部子集为：


```text
2^n
```


正式实验中可能有 10、20、30+ 个 action instances，因此不能穷举。

PatchCredit 要在固定 execution budget 下选择最有价值的反事实计划，例如：

- 优先高不确定性动作；
- 优先疑似 interaction 边；
- 优先对当前最优重组决策影响大的计划；
- 跳过明显非法的 pipeline；
- 根据历史构建/运行失败概率降低无效采样；
- 分层筛选：先粗筛，再精细归因。

目标不是“提出一个新采样器名字”，而是：

> **在相同真实编译/运行预算下，比 random、LOO、Shapley sampling、简单 bandit/回归基线恢复更有用的 credit/interaction，并找到更好的最终计划。**

---

## 支撑创新 2：Credit-guided Recomposition

归因不是最终输出。

PatchCredit 必须利用 credit 与 interaction 构造新的优化计划：


```text
S_LLM -> S_PC
```


预实验阶段优先做：

1. order-preserving pruning；
2. block removal；
3. 保留高价值 synergy group；
4. 去除负贡献/高冲突动作。

正式阶段再增加：

5. 对相互独立动作进行局部 reorder；
6. 对重复 Pass instance 做保留/移除决策；
7. 在合法 grammar 下搜索 nested pipeline 的局部结构。

核心成功条件：


```text
Runtime(S_PC) < Runtime(S_LLM)
```


或者在 runtime 几乎不损失时显著减少计划复杂度和执行成本。

---

# 5. 它和现有工作的边界

## Compiler-R1

Compiler-R1 研究：

> LLM + RL 如何生成更好的 compiler pass sequence。

它还构建了 pass synergy graph，因此 PatchCredit 不能声称“首次研究 Pass 协同”。

PatchCredit 的区别是：

> **不负责从零生成 sequence，而是对已经生成的具体 sequence 做 post-hoc、runtime-based、contextual counterfactual credit assignment，再重组。**

参考：
- https://arxiv.org/abs/2506.15701
- https://github.com/Mind4Compiler/Compiler-R1

## ECCO

ECCO 用性能证据和因果推理指导 LLM/GA 搜索 pass sequence。

PatchCredit 不能声称“首次用因果证据做 compiler optimization”。

区别：

> ECCO：evidence → strategist → search  
> PatchCredit：generated plan → counterfactual attribution → recomposition

参考：
- https://arxiv.org/abs/2602.00087

## Synergy-Guided Compiler Auto-Tuning

该工作已经显式挖掘 synergistic pass relationships，并用它指导 LLVM New Pass Manager 的搜索。

PatchCredit 不能把“发现协同 Pass pair”当主创新。

区别：

> PatchCredit 的 interaction 是**计划内、上下文条件化、基于真实 runtime 干预**，并用于对具体 LLM plan 进行 credit assignment。

参考：
- https://arxiv.org/abs/2510.13184

## Per-Pass LLVM Empirical Study

2026 年 per-pass empirical study 已经系统测量 LLVM `-O3` pipeline 的 cumulative prefixes，研究单 Pass 边际影响、噪声、性能回退和 phase interference。

PatchCredit 不能声称“首次量化 individual pass runtime contribution”。

区别：

> 该研究针对固定 `-O3` pipeline 的 prefix；PatchCredit 针对任意 LLM-generated plan 的反事实删除/组合上下文。

参考：
- https://arxiv.org/abs/2606.31238

## AutoPass

AutoPass 已经用 compiler/runtime evidence 反馈给 LLM agents，并迭代修改 optimization decisions。

因此 PatchCredit 不能退化成：

> “跑一下 runtime，发现某 Pass 不好，再让 LLM 改。”

PatchCredit 必须保留独立的 attribution 层：

> **用有限的反事实真实执行恢复具体动作的 contextual credit，再由算法重组。**

参考：
- https://arxiv.org/abs/2606.20373

## TRIM

TRIM 主要面向 agent-generated source patch 的层次化反事实删除和最小化。

它现在不是 PatchCredit 的最直接编译器竞品，但仍提醒我们：

> “删除动作 + 执行验证 + 最小化”本身不是新颖性。

参考：
- https://arxiv.org/abs/2607.18161

---

# 6. 最小可实现版本建议

为了避免第一版工程过大，建议 V1 严格锁定：

- 编译器：LLVM；
- 对象：线性 pass sequence / 可序列化的 New Pass Manager pipeline；
- 指标：真实 runtime 为主；
- 正确性：输出/benchmark correctness；
- LLM：负责生成初始优化计划；
- PatchCredit：负责归因与二次重组；
- 预实验动作数：4–8；
- 正式动作数：10–30+；
- 预实验只做删除型 intervention；
- 正式实验再考虑局部 reorder。

### V1 暂时不做

- 不训练新的大模型；
- 不一开始做多架构；
- 不直接修改 LLVM pass implementation；
- 不做任意源代码 diff attribution；
- 不一开始做复杂 nested pipeline grammar；
- 不把 LLM 生成解释文本当作性能证据。

---

# 7. 数据从哪里来

PatchCredit 的主数据必须是：

> **LLM Compiler Agent 实际生成的优化计划。**

推荐数据链：

```text
Benchmark Program
      ↓
统一 baseline 编译和 runtime
      ↓
LLM Compiler Agent
      ↓
生成 S_LLM
      ↓
编译
      ↓
correctness
      ↓
重复 runtime
      ↓
筛出“正确 + 可测加速 + 多动作”的计划
      ↓
PatchCredit Dataset
```

### 预实验候选环境

1. **Compiler-R1 兼容的 LLVM pass-sequence 任务**
   - 可复用其代码/数据结构/Pass 搜索接口；
   - 但 PatchCredit 的主指标应切到 runtime，而不是只看 IR instruction count。
   - https://github.com/Mind4Compiler/Compiler-R1

2. **CompilerGym LLVM 环境**
   - 提供 LLVM 优化 action 环境和多个 benchmark；
   - 适合快速构造 agent → pass sequence 数据链。
   - https://github.com/facebookresearch/CompilerGym

3. **PolyBench/C / llvm-test-suite**
   - 适合做稳定 runtime 测量和 correctness；
   - 2026 per-pass study 已证明 PolyBench/C 可用于严格的 pass runtime 实验。

4. **PassBench**
   - 200 个 graph compiler pass generation 任务；
   - 更适合后续外部扩展，而不是第一版 LLVM pass-sequence 主实验。
   - https://arxiv.org/abs/2605.29357

---

# 8. 预实验最重要的科学问题

不要先问：

> “PatchCredit 能不能做到 SOTA？”

先问四件事：

### H1：LLM 生成的成功计划里是否存在明显负贡献或冗余 action？

如果几乎没有，重组空间有限。

### H2：是否存在稳定的 context-dependent interaction？

如果 action contribution 基本独立，复杂 credit 模型没有必要。

### H3：完整 LLM plan 是否经常不是局部最优？

如果完整计划本身已经是最佳子集，PatchCredit 很难产生额外性能收益。

### H4：用少量实验能否恢复足够有用的结构？

如果 random/LOO 已经足够，复杂 budgeted estimator 不值得做。

---

# 9. 论文最终要回答的核心问题

正式论文至少回答：

- RQ1：LLM optimization plan 中是否存在稳定的上下文性能归因结构？
- RQ2：PatchCredit 的 contextual credit 是否比 prefix/LOO 更能预测动作重要性？
- RQ3：预算化采样能否减少真实编译/运行次数？
- RQ4：credit-guided recomposition 能否超过原始 LLM plan？
- RQ5：结果在不同程序/不同 plan 长度下是否稳定？
- RQ6：runtime improvement 与 IR instruction count 是否一致？不一致时 PatchCredit 能否捕捉真实 runtime 差异？

---

# 10. 当前成功标准

## 预实验 GO

在至少 8–12 个可穷举的真实 LLM compiler plans 上，满足以下大部分条件：

- 明显存在 negative/redundant actions；
- 至少约 30% 的计划存在可重复的非加性交互；
- 至少约 20% 的计划存在比完整 LLM plan 更好的合法子计划，或明显 Pareto 改善；
- PatchCredit-Lite 能在明显低于穷举预算下找回接近 Oracle 的方案；
- runtime 测量噪声低于主要 attribution effect。

这些比例是**预注册式工程 Gate**，不是已有实验结果，可在首轮 smoke test 后根据测量噪声做一次冻结式调整。

## 正式论文目标

优先级：

1. **最终 runtime 改善**；
2. execution budget 降低；
3. attribution/interaction 稳定性；
4. plan compactness；
5. 编译时间和 code size 作为辅助指标。

---

# 11. 当前风险

### 风险 1：LLM 成功 runtime plan 太少
应对：
- 每个 benchmark 多次生成；
- 允许多模型/多 agent scaffold；
- 使用已有 Compiler-R1 环境做初始 sequence 生成；
- 先筛容易稳定测量的程序。

### 风险 2：Plan 太长导致组合爆炸
应对：
- 预实验只筛 4–8 action；
- 正式版做 hierarchical screening + budgeted sampling。

### 风险 3：Runtime 噪声淹没 credit
应对：
- CPU pinning；
- warm-up；
- 多次重复；
- robust aggregation；
- CI；
- 把低信号任务踢出主数据。

### 风险 4：最后只是 LOO
如果 LOO 已经足够找到最优重组，则主创新不足。
必须通过 P5/P6 Gate 决定是否继续。

### 风险 5：最后变成普通 phase ordering
必须始终保持研究问题：

> **不是从零搜索最优 sequence，而是解释和修正 LLM 已经生成的 plan。**

---

# 12. 当前一句话摘要

> **PatchCredit 是一个面向 LLM Compiler Agent 的事后性能归因与二次优化框架：它通过真实执行反事实实验估计优化计划中各动作的上下文性能贡献和交互，在有限测量预算下识别负贡献、协同和冗余动作，并据此重组出正确且更高性能的编译优化计划。**

