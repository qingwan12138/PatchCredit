# PatchCredit 新版创新性审查结论（详细版）

## 结论

当前框架的危险不是“已经被某一篇论文完全做完”，而是：

> **每一个单点都已经有强近邻。**

因此 PatchCredit 只能依赖组合边界：

```text
LLM-generated plan
→ post-hoc
→ action-instance
→ contextual
→ real-runtime counterfactual attribution
→ budgeted interventions
→ recomposition
```

---

# 1. 最大威胁

## Compiler-R1

覆盖：
- LLM；
- RL；
- pass-sequence auto-tuning；
- synergy。

所以：
- Plan generation 不是新；
- Pass synergy 不是新。

## ECCO

覆盖：
- evidence；
- causal reasoning；
- compiler optimization；
- LLM strategist + search。

所以：
- causal/evidence 不能当核心 novelty。

## AutoPass

覆盖：
- compiler/runtime evidence；
- LLM agent iterative tuning。

所以：
- “根据 runtime 反馈让 LLM 改”不是新。

## Per-Pass LLVM Study

覆盖：
- fixed O3 pipeline；
- cumulative prefix effects；
- runtime；
- noise；
- phase interference。

所以：
- “量化 Pass Runtime 贡献”不是新。

---

# 2. 仍可守的技术空白

当前可继续验证的空白是：

> 对 LLM 已经生成的具体 Plan 做局部反事实 intervention，估计 action instance 在当前 plan 邻域的 contextual runtime credit，并在有限测量预算下用该信息修复 Plan。

这里至少同时要求：

```text
specific generated plan
instance-level
context-conditioned
runtime-based
counterfactual
budget-aware
decision-useful
```

---

# 3. 最大的非论文风险：LOO

即使没有强论文完全撞车，框架也可能被一个极简单方法击败：

```text
for each action:
    remove action
    rerun
rank actions by delta
greedily prune
```

因此：

> **LOO 是预实验最重要的 baseline。**

如果 LOO 已经得到几乎全部 extra speedup：

- 主创新 1 需要降级；
- 支撑创新 1 可能失去必要性。

---

# 4. 当前研究评分（预实验前）

| 维度 | 内部评价 |
|---|---:|
| LLM Compiler 相关性 | 9/10 |
| 问题清晰度 | 8.5/10 |
| 单点新颖性 | 6/10 |
| 组合新颖性 | 7.5–8/10 |
| 可实现性 | 8/10 |
| Benchmark 可获得性 | 8.5/10 |
| Runtime 实验风险 | 中 |
| 竞品压力 | 高 |
| 是否值得做 Pilot | 是 |

该评价不是录用概率。

---

# 5. 必须通过的预实验事实

继续正式开发前，至少要看到：

```text
1. LLM 能生成足够的正确且加速的多动作 Plan
2. Plan 内存在稳定 negative/redundant action
3. Interaction 不是罕见孤例
4. Full LLM Plan 经常不是最佳局部子计划
5. LOO 不能完全解释/解决问题
6. 少量采样有机会恢复 Oracle 结构
7. Recomposition 能在 clean rerun 中真实提高 Runtime
```

---

# 6. 结论标签

```text
Novelty status:
CONDITIONAL PASS

Reason:
No full direct collision found,
but all individual ingredients have strong neighbors.

Required next action:
Go/No-Go prestudy before any large engineering investment.
```
