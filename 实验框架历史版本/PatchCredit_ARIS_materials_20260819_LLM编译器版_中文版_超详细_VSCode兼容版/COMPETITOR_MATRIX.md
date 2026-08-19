# PatchCredit 详细竞品矩阵与创新边界

> 版本：2026-08-19  
> 目标：避免把已有工作重新命名成“创新”。

---

# 1. 新颖性必须守住的组合

PatchCredit 当前的安全主张不是任何一个单点，而是：

```text
LLM-generated compiler optimization plan
+
action-instance level
+
post-hoc contextual counterfactual runtime credit
+
budgeted intervention selection
+
credit-guided recomposition
```

任何一个单项都可能已有强近邻。

---

# 2. 强竞品总览

| 工作 | 类别 | 已覆盖能力 | 与 PatchCredit 重叠 | 威胁等级 | PatchCredit 安全差异 |
|---|---|---|---|---|---|
| Compiler-R1 | LLM compiler auto-tuning | LLM+RL 生成 LLVM pass sequence，synergy graph | plan generation、pass synergy | 很高 | post-hoc contextual runtime attribution |
| ECCO | evidence-driven compiler optimization | performance evidence、causal reasoning、GA search | causal/evidence、search | 很高 | 生成计划之后的 credit assignment |
| Synergy-Guided Auto-Tuning | phase ordering | synergy mining + LLVM NPM search | interaction | 高 | instance-level context-conditioned interaction |
| AutoPass | LLM compiler agent | runtime/compiler evidence + iterative tuning | runtime feedback、LLM repair | 很高 | explicit intervention/credit model |
| Per-Pass LLVM Study | empirical compiler study | fixed O3 prefix per-pass effects、noise、phase interference | pass contribution | 很高 | arbitrary LLM plan + non-prefix interventions |
| CompilerGym | infrastructure | LLVM action environment、benchmarks、runtime observations | experiment platform | 不是竞品 | 作为实验基础设施 |
| TRIM | agent patch minimization | hierarchical counterfactual deletion | deletion/minimization | 中 | compiler plan + runtime credit, not patch minimality |
| LOO / Shapley | attribution baseline | contribution estimation | credit assignment | 高算法基线 | context/constraints/budget + final recomposition |
| Delta Debugging | minimization | failure-preserving subset minimization | subset deletion | 中 | performance utility + interaction |
| Traditional Phase Ordering | compiler tuning | sequence search | recomposition/search | 高 | not search-from-scratch; LLM-plan diagnosis |

---

# 3. Compiler-R1

参考：
- https://arxiv.org/abs/2506.15701
- https://github.com/Mind4Compiler/Compiler-R1

官方项目说明其目标是：

```text
LLM + RL
→ compiler pass sequence auto-tuning
→ reduce LLVM IR instruction count
```

并且其工作已经包含 pass synergy 相关结构。

因此下面这些都不能写：

```text
“首次用 LLM 生成 compiler pass sequence”
“首次建模 pass synergy”
“首次让 LLM 和 compiler environment 交互”
```

PatchCredit 应该写：

```text
Compiler-R1:
generate/search a plan

PatchCredit:
diagnose and repair an already-generated plan
using real runtime counterfactuals
```

---

# 4. ECCO

PatchCredit 与 ECCO 最危险的重合词：

```text
causal
evidence
compiler optimization
LLM
search
```

因此论文中尽量不要泛化写：

> “我们首次引入因果推理解释编译器优化。”

更准确：

> “我们对 LLM 已生成的具体优化计划执行 action-level counterfactual runtime interventions，并估计局部上下文 Credit。”

---

# 5. AutoPass

如果 PatchCredit 最后实现成：

```text
run plan
→ 找 bad pass
→ prompt LLM
→ LLM regenerate
```

则很危险。

PatchCredit 必须输出可独立评价的中间对象：

```text
credit table
interaction table
uncertainty
measured intervention set
candidate ranking
recomposition trace
```

这样可以做独立 RQ：

> attribution 本身准不准？

而不只是看最终 speedup。

---

# 6. Per-Pass LLVM Study

该类研究已经证明：

- fixed optimization pipeline 中存在 non-monotonic transitions；
- phase interference 是现实问题；
- runtime measurement 需要严格控制噪声。

所以 PatchCredit 主创新不能是：

> “我们发现某些 pass 会让程序变慢。”

真正要做的是：

> **在 LLM-generated arbitrary plan 上，为具体 action instance 恢复 context-conditioned contribution，并用于决策。**

---

# 7. LOO 是必须重点防的算法基线

PatchCredit 最大的“简单方法风险”其实不是论文竞品，而是：

```text
LOO deletion
```

如果对一个 20-action Plan：

```text
只跑 full plan + 20 个 leave-one-out plans
```

就能：

- 找到负贡献动作；
- 找到最佳重组；
- 获得几乎全部最终收益；

那么复杂 Contextual Credit / Budgeted Sampling 都没必要。

因此预实验必须专门设计：

```text
PatchCredit vs LOO
```

而且同 execution budget 比。

---

# 8. Shapley / Monte Carlo Shapley

Shapley 能定义多动作平均边际贡献，但问题是：

- execution cost 高；
- compiler pipeline 可能非法；
- action effect 强依赖位置与 IR 状态；
- 全局平均贡献可能掩盖当前 Plan 局部上下文。

PatchCredit 需要证明：

> 局部、约束感知、决策导向 Credit 对“修当前 LLM Plan”更有效。

不能只说 Shapley 贵。

---

# 9. 传统 Phase Ordering

传统 phase ordering 从零搜索：

```text
search space
→ sequence
→ evaluate
→ search
```

PatchCredit 的合理实验必须固定一个前提：

```text
S_LLM 已经存在
```

然后比较：

```text
same execution budget
PatchCredit local repair
vs
random local search
vs
genetic local search
vs
greedy pruning
```

如果 PatchCredit 必须从零搜索才能有效，论文定位就会塌回普通 phase ordering。

---

# 10. 可安全主张 / 不可安全主张

## 相对安全

- 面向 LLM-generated compiler plans 的 post-hoc contextual credit；
- action-instance 而非 pass-type 级归因；
- 真实 runtime counterfactual interventions；
- budgeted intervention selection for attribution；
- credit-guided local recomposition；
- runtime-proxy disagreement analysis。

## 不安全

- 首次 LLM compiler tuning；
- 首次 pass interaction；
- 首次 performance evidence；
- 首次 causal compiler optimization；
- 首次 per-pass runtime contribution；
- 首次 runtime feedback；
- 首次 counterfactual deletion；
- 首次 minimal plan。

---

# 11. 正式实验必须覆盖的竞品维度

| 论文主张 | 必须基线 |
|---|---|
| Credit 更准 | Prefix、LOO、Monte Carlo Shapley |
| Interaction 更有用 | Pairwise LOO、main+pairwise regression |
| Sampling 更省 | Random、LOO-first、Shapley sampling |
| Recomposition 更强 | Greedy prune、random local search、same-budget search |
| Compiler 效果 | -O2、-O3、原始 LLM Plan |
| LLM 价值 | 至少说明初始 Plan 确实由 LLM/Agent 产生 |

---

# 12. 新颖性生死线

如果预实验发现：

```text
LOO ≈ PatchCredit
且
interaction 很少
且
recomposition 无额外 runtime gain
```

则：

> 不应通过“换更复杂模型”强行保住主创新。

如果发现：

```text
same action has different effect under nearby contexts
+
pairwise / higher-order effects matter
+
LOO misses good recomposition
+
budgeted local interventions recover it
```

则主创新成立概率明显提高。
