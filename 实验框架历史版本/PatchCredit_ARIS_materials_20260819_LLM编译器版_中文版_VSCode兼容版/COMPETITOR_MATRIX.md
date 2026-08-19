# COMPETITOR MATRIX：PatchCredit（LLM 编译器版）

> 日期：2026-08-19  
> 状态：**强近邻复核完成；尚未运行 Pilot**  
> 查新口径：LLM-generated compiler optimization plan 的 post-hoc contextual counterfactual runtime credit assignment + budgeted attribution + recomposition。

---

# 1. 当前最强竞品结论

当前没有发现一个工作完整覆盖以下四件事：

1. 输入是 **LLM Compiler Agent 已生成的具体优化计划**；
2. 对计划中的 action instances 做 **真实 runtime 的上下文反事实 credit assignment**；
3. 在有限 execution budget 下主动选择干预计划；
4. 用 credit/interaction 结果对原计划进行 **二次重组并实测**。

但是存在四组强近邻：

- Compiler-R1 / Synergy-Guided：LLM/pass sequence + synergy；
- ECCO：evidence/causal reasoning + compiler search；
- AutoPass：LLM + runtime evidence feedback；
- Per-Pass LLVM Study：individual pass runtime contribution + interference。

因此 PatchCredit 的新颖性必须建立在**四者交叉空白**上，而不能用单个关键词主张创新。

---

# 2. 竞品总表

| 工作 | 年份 | 已经做了什么 | 对 PatchCredit 的威胁 | PatchCredit 必须守住的区别 |
|---|---:|---|---|---|
| Compiler-R1 | 2025 | LLM+RL 做 LLVM pass sequence auto-tuning；构建 Global Synergy Graph | **高** | 不做“首次 LLM pass sequence / 首次 pass synergy”；聚焦具体生成计划的 post-hoc contextual runtime credit |
| Synergy-Guided Compiler Auto-Tuning | 2025 | 挖掘 synergistic pass relationships，指导 LLVM NPM 搜索 | **高** | interaction 必须是 plan-conditioned + intervention-based + runtime-based |
| ECCO | 2026 | 性能 evidence + causal reasoning，让 LLM strategist 指导 GA 搜索 | **很高** | 不声称首次 causal/evidence compiler optimization；PatchCredit 在生成计划之后做 attribution |
| AutoPass | 2026 | compiler/runtime evidence 指导 LLM agents 反复做性能 tuning | **很高** | 不能只是“反馈 runtime 再改”；必须有独立可评估的 counterfactual credit layer |
| Per-Pass LLVM Empirical Study | 2026 | 113 个 `-O3` cumulative prefixes，84,750 次测量，分析 runtime/energy/phase interference | **很高** | 不声称首次 pass contribution；PatchCredit 面向任意 LLM-generated plan 的非-prefix interventions |
| TRIM | 2026 | agent source patch 的层次化反事实删除和最小化 | 中 | 删除/最小化不是创新；PatchCredit 的对象和 utility 是 compiler plan + runtime credit |
| PassNet / PassBench | 2026 | LLM 生成 graph compiler passes；200 tasks | 中 | 可作为外部 compiler-domain 验证，不覆盖 plan attribution |
| CompilerGym | 持续维护 | LLVM action environment / phase-ordering 实验基础设施 | 基础设施 | 不作为竞品创新；可做实验平台 |
| AutoPhase 等 phase ordering | 经典 | RL/ML 搜索 pass sequence | 中 | PatchCredit 不是从零 phase ordering，而是 post-hoc repair/recomposition |
| LOO / Shapley / sparse regression | 通用 | 贡献归因基础方法 | **强算法基线** | 必须同预算比较，不能只和 random 比 |

---

# 3. 逐项边界

## 3.1 Compiler-R1

参考：
- https://arxiv.org/abs/2506.15701
- https://github.com/Mind4Compiler/Compiler-R1

Compiler-R1 已经覆盖：

- LLM 进行 pass sequence auto-tuning；
- RL；
- environment interaction；
- pass pair synergy；
- Global Synergy Graph；
- 多 benchmark 评测。

因此禁止主张：

- “首次让 LLM 优化 pass sequence”；
- “首次发现 pass 之间存在协同”；
- “首次使用 graph 表示 pass synergy”。

PatchCredit 的边界：

> 对 **已生成的具体 sequence** 做真实 runtime intervention，估计 action instance 在当前 sequence 上下文中的 credit，并用于二次重组。

---

## 3.2 ECCO

参考：
- https://arxiv.org/abs/2602.00087

ECCO 已经覆盖：

- static code feature；
- verifiable performance evidence；
- causal reasoning；
- LLM strategist；
- GA search；
- compiler optimization。

因此禁止主张：

- “首次 evidence-driven compiler optimization”；
- “首次 causal compiler reasoning”；
- “首次用 performance evidence 帮助 LLM 搜索”。

PatchCredit 的区别：

```text
ECCO:
features/evidence → LLM strategist → GA search → sequence

PatchCredit:
LLM/agent → sequence → counterfactual runtime attribution → recomposition
```

---

## 3.3 Synergy-Guided Compiler Auto-Tuning

参考：
- https://arxiv.org/abs/2510.13184

已覆盖：

- valid nested LLVM New Pass Manager pipelines；
- formal grammar；
- synergy mining；
- synergy-guided GA；
- structural refinement。

因此 PatchCredit 不能把：

> “pass pair synergy mining”

单独作为主创新。

必须升级为：


```text
Interaction(a_i, a_j | S)
```


即：

> **具体计划内、上下文条件化、动作实例级、基于 runtime 干预的 interaction。**

---

## 3.4 Per-Pass LLVM Empirical Study

参考：
- https://arxiv.org/abs/2606.31238

已覆盖：

- LLVM `-O3`；
- 113 cumulative prefixes；
- 30 PolyBench/C kernels；
- 84,750 measurements；
- execution time、compile time、binary size、hardware counters、energy；
- non-monotonic transitions；
- phase interference；
- noise mitigation。

因此禁止主张：

> “首次测每个 Pass 对 runtime 的贡献。”

PatchCredit 的区别：

- 不限于固定 `-O3`；
- 不限于 prefix；
- intervention 可删除任意 action instance；
- 研究局部上下文 credit；
- 研究 LLM-generated plans；
- 最终做 recomposition。

---

## 3.5 AutoPass

参考：
- https://arxiv.org/abs/2606.20373

AutoPass 已覆盖：

- LLM agents；
- compiler evidence；
- runtime evidence；
- 真实性能反馈；
- 迭代 tuning。

PatchCredit 最大风险：

如果最终实现只是：

```text
run sequence
→ 找一个回退动作
→ 告诉 LLM
→ LLM 再给 sequence
```

则创新边界非常危险。

必须做成：

```text
S_LLM
→ explicit intervention set
→ measured utility table
→ contextual credit/interaction model
→ algorithmic recomposition
→ S_PC
```

LLM 可以辅助：
- 初始 plan generation；
- action semantic summary；
- interaction hypothesis。

但 credit 不能由 LLM 主观判断。

---

## 3.6 TRIM

参考：
- https://arxiv.org/abs/2607.18161

TRIM 提醒：

> hierarchical deletion + execution validation + minimal plan/patch 不是新颖性。

所以 PatchCredit 的目标不是“越短越好”，而是：


```text
maximize: Performance(S) - lambda * Cost(S)
```


其中 performance 第一优先级。

---

## 3.7 PassNet / PassBench

参考：
- https://arxiv.org/abs/2605.29357

PassNet 将 LLM 直接用于 graph compiler pass generation，并公开 PassBench 200 tasks。

它不是最直接的 sequence-attribution 竞品，但说明：

> “LLM + compiler pass”本身已经成为明确方向。

PatchCredit 后续可以把它作为跨 compiler abstraction 的外部验证：

- LLVM pass sequence；
- graph compiler generated pass。

但不建议第一版同时做两个体系。

---

# 4. 主创新的最低新颖性要求

PatchCredit 主创新必须同时满足：

1. **Object specificity**  
   研究对象是 LLM-generated compiler plan，而非固定 O3 pipeline。

2. **Action-instance level**  
   同一 Pass 在不同位置/参数下视为不同实例。

3. **Contextuality**  
   不输出全局 `Credit(PassType)`，而输出 `Credit(a_i | S)`。

4. **Counterfactual execution**  
   Credit 由真实可执行 compiler interventions 支撑，而不是 LLM 自述。

5. **Runtime utility**  
   核心 utility 是真实 runtime，不把 IR instruction count 当最终性能代理。

6. **Budget awareness**  
   不能默认穷举。

7. **Decision usefulness**  
   Credit 必须用于 recomposition，并最终用实际性能验证。

如果缺任意三项以上，创新性会明显下降。

---

# 5. 强基线清单

## 归因基线

- Prefix marginal contribution；
- LOO deletion；
- Pairwise deletion；
- Monte Carlo Shapley sampling；
- sparse linear model；
- pairwise regression；
- random forest / GBDT interaction surrogate（可选）。

## 采样基线

- Random subset；
- Uniform size-stratified subset；
- LOO-only；
- Shapley random permutation；
- uncertainty-only；
- greedy deletion / ddmin-style。

## 重组基线

- 原始 LLM plan；
- greedy remove-negative；
- LOO-ranked pruning；
- random local search；
- genetic/local search with same execution budget；
- synergy-only heuristic。

## 编译优化强基线

- `-O2` / `-O3`；
- Compiler-R1-style generated plan；
- 若能复现：ECCO / AutoPass 的公开或近似设置；
- Synergy-guided search。

---

# 6. 当前风险评级

| 风险 | 等级 | 原因 |
|---|---:|---|
| “Pass contribution”已有人做 | 高 | Per-Pass Study |
| “Pass synergy”已有人做 | 高 | Compiler-R1 + Synergy-Guided |
| “runtime evidence + LLM loop”已有人做 | 高 | AutoPass |
| “causal/evidence compiler optimization”已有人做 | 高 | ECCO |
| “LLM compiler pass generation”已有人做 | 高 | Compiler-R1 / PassNet |
| “contextual post-hoc credit of generated plan”直接竞品 | 当前中低 | 暂未发现完整覆盖 |
| “budgeted attribution + recomposition”直接竞品 | 当前中低 | 暂未发现完整覆盖 |

---

# 7. 当前建议

继续做，但论文标题、摘要、创新点不要写成：

> LLM pass optimization / pass synergy / performance attribution

这种过宽表述。

更安全的核心描述：

> **Contextual Counterfactual Credit Assignment for LLM-Generated Compiler Optimization Plans**

中文：

> **面向 LLM 编译优化计划的上下文反事实性能归因与预算化重组。**

