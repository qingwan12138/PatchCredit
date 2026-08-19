# PatchCredit 新版创新性审查请求（详细版）

## 1. 审查对象

研究框架：

> 面向 LLM Compiler Agent 已生成的多动作优化计划，执行真实 compiler/runtime 反事实干预，估计 action-instance 级上下文性能 Credit 与 Interaction；在不能穷举子计划时使用预算感知采样；最终根据 Credit 进行二次重组。

---

# 2. 待审查主张 A1：Contextual Credit

输入：

```text
S_LLM = [a_1, ..., a_n]
```

输出：

```text
Credit(a_i | S_LLM)
Interaction(a_i, a_j | S_LLM)
```

要求：

- action instance 与位置、参数、pipeline context 绑定；
- utility 来自真实 runtime；
- compile/correctness failure 视为 infeasible，不作为正常 runtime；
- 不退化成 fixed O3 prefix contribution；
- 不退化成单次 LOO。

---

# 3. 待审查主张 A2：Budgeted Intervention Selection

目标：

```text
在 B 次真实编译/运行预算内
恢复足够准确的 Credit/Interaction
以支持最终重组
```

候选信号：

- uncertainty；
- decision impact；
- interaction potential；
- expected execution cost；
- invalid plan risk。

必须与：

- random；
- LOO-first；
- Shapley sampling；
- greedy deletion；

做同预算比较。

---

# 4. 待审查主张 A3：Credit-Guided Recomposition

第一阶段只做：

```text
order-preserving deletion
block pruning
synergy retention
conflict removal
```

正式扩展：

```text
local reorder
repeated-pass instance handling
nested local rewrite
```

最终必须：

```text
Runtime(S_PC) < Runtime(S_LLM)
```

或在 Runtime 基本不损失时显著降低 Plan 复杂度/调优成本。

---

# 5. 明确不主张

不主张：

- 首次 LLM pass auto-tuning；
- 首次 pass synergy；
- 首次 compiler causal reasoning；
- 首次 performance evidence；
- 首次 individual pass runtime contribution；
- 首次 runtime-feedback LLM agent；
- 首次 counterfactual deletion/minimization。

---

# 6. 必查竞品

- Compiler-R1
- ECCO
- Synergy-Guided Compiler Auto-Tuning
- AutoPass
- Per-Pass LLVM empirical study
- TRIM
- CompilerGym related phase-ordering work
- LOO
- Shapley
- Delta Debugging
- traditional phase ordering

---

# 7. 判定标准

若已有单一工作同时完成：

```text
LLM-generated plan
+
action-instance contextual runtime attribution
+
budgeted counterfactual selection
+
credit-guided recomposition
```

则主框架判严重撞车。

若只覆盖若干组件，则：

- 列出重合；
- 限制可用 novelty statement；
- 给出必须保留的技术差异；
- 给出预实验必须击败的简单 baseline。
