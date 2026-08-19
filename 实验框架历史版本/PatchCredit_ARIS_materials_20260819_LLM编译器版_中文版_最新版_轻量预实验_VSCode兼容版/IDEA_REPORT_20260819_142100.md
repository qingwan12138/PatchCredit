# PatchCredit：LLM 编译优化计划的上下文反事实性能归因与二次优化
## 详细研究报告（VS Code 兼容版）

> 版本：2026-08-19  
> 状态：研究设计冻结候选版；尚未通过预实验 Go/No-Go  
> 研究方向：LLM + Compiler Auto-Tuning / Compiler Optimization  
> 核心目标：**真实 Runtime 性能提升**  
> 核心对象：**LLM Compiler Agent 生成的多动作编译优化计划**

---

# 1. 一句话定义

PatchCredit 做的事情可以概括为：

```text
LLM Compiler Agent 先生成一个优化计划
        ↓
真实编译、正确性验证、Runtime 测量
        ↓
PatchCredit 判断“计划里的每个动作到底贡献了什么”
        ↓
识别负贡献 / 冗余 / 协同 / 冲突
        ↓
只测少量最有价值的反事实组合
        ↓
重新组合优化动作
        ↓
得到比原 LLM Plan 更快或更紧凑的 Plan
```

目标不是“再让 LLM 猜一次”，而是：

> **用真实可执行实验恢复 LLM 优化计划内部的性能贡献结构，再根据这个结构做二次优化。**

---

# 2. 为什么现在研究对象不是 Source Code Patch

早期 PatchCredit 版本研究的是：

```text
LLM 修改源代码
→ 多个 diff/edit clusters
→ 做性能归因
→ 重组源代码 Patch
```

这个版本存在两个问题：

1. 容易被归类到 LLM Software Engineering / Performance Engineering；
2. 和 agent patch minimization、code optimization benchmark 的边界不够纯粹。

当前版本收紧为：

```text
程序 / LLVM IR
→ LLM Compiler Agent
→ Compiler Optimization Plan
→ PatchCredit
```

所以论文核心属于：

> **LLM Compiler Auto-Tuning 的后验归因与二次优化。**

---

# 3. 研究对象的严格定义

给定程序 x，LLM Compiler Agent 生成：

```text
S_LLM = [a_1, a_2, ..., a_n]
```

每个 a_i 不是抽象“Pass 类型”，而是一个 **Action Instance**：

```text
a_i = {
    pass_name,
    position,
    parameters,
    nesting_context,
    input_ir_state_hash,
    source_agent
}
```

例如：

```text
a_3 = {
    pass_name: "loop-unroll",
    position: 3,
    parameters: {"count": 4},
    nesting_context: "function(loop(...))",
    input_ir_state_hash: "...",
    source_agent: "llm_agent_v1"
}
```

这样定义的原因：

- 同一个 Pass 出现在不同位置，作用可能完全不同；
- Pass 的效果依赖之前的 IR 状态；
- 参数不同会改变效果；
- New Pass Manager 可能存在 nested pipeline；
- PatchCredit 研究的是“这个动作在这个具体计划中的贡献”，而不是给 Pass 类型打全局分数。

---

# 4. PatchCredit 最终输出什么

输入：

```text
Program x
S_LLM = [A, B, C, D, E, F]
Runtime(S_LLM) = 80 ms
```

输出包括四类信息。

## 4.1 Action Credit

```text
Credit(A | S_LLM)
Credit(B | S_LLM)
...
```

例如：

```text
A: +4.1 ms benefit
B: +0.7 ms benefit
C: -2.8 ms benefit
D: approximately 0
E: context-dependent
F: +1.9 ms benefit
```

负值表示在当前上下文中可能拖慢。

## 4.2 Interaction

```text
Interaction(B, E | S_LLM) = strong positive
Interaction(C, F | S_LLM) = negative conflict
```

## 4.3 Uncertainty

每个 Credit / Interaction 必须有：

```text
mean
confidence interval
uncertainty score
number of supporting measurements
```

## 4.4 Recomposed Plan

例如：

```text
S_PC = [A, B, E, F]
Runtime(S_PC) = 73 ms
```

最终真正有价值的是：

```text
Runtime(S_PC) < Runtime(S_LLM)
```

---

# 5. 三个创新点

# 创新点 1（主创新）
## Contextual Counterfactual Credit Assignment

普通做法可能只计算：

```text
prefix_delta(i)
或
LOO(i) = U(S) - U(S_without_i)
```

PatchCredit 要研究的是：

```text
Credit(a_i | current_plan_context)
```

即：

> 同一个 action 在不同上下文中，其真实性能贡献可以不同。

反事实基本形式：

```text
Delta_i(S) = U(S) - U(S_without_a_i)
```

但正式方法不能只做单次 LOO，而要在 S 附近多个**可行子上下文**中进行估计：

```text
C_i ≈ average[
    U(context + a_i) - U(context)
]
```

其中 context 必须满足：

- compiler pipeline 合法；
- build 成功；
- correctness 通过；
- runtime 可稳定测量。

### 主创新最终要证明

不是只证明：

> “我们能删 Pass。”

而是证明：

> **Contextual Credit 比 Prefix / LOO 更能解释并预测一个具体 LLM Plan 里的动作重要性。**

---

# 创新点 2（支撑创新）
## Budgeted Counterfactual Credit Estimation

如果 plan 有 20 个 action：

```text
2^20 = 1,048,576 subsets
```

如果 plan 有 30 个 action：

```text
2^30 > 1 billion subsets
```

不可能全部真实编译运行。

所以第二创新解决：

> **下一次最值得测哪个反事实 Plan？**

每个候选 intervention 计算一个采样价值：

```text
Acquisition =
    uncertainty
  + decision_impact
  + interaction_potential
  - expected_execution_cost
  - invalid_plan_risk
```

优先测：

- 当前 Credit 最不确定的动作；
- 疑似强协同/冲突的动作对；
- 会改变最终重组选择的组合；
- 有较高编译/正确性成功概率的组合；
- coarse screening 后留下的关键 action/block。

### 第二创新最终要证明

在相同真实执行预算下：

```text
PatchCredit
vs
Random
vs
LOO-only
vs
Shapley sampling
vs
Greedy deletion
```

至少做到一项：

- 更快找到接近 Oracle 的 Plan；
- 更低 regret；
- 更高 top-k action recovery；
- 更高 strong-interaction recall；
- 用更少 executions 达到同等结果。

---

# 创新点 3（支撑创新）
## Credit-Guided Recomposition

归因不是论文终点。

PatchCredit 必须利用归因做：

```text
negative action removal
redundant action pruning
synergy group retention
conflict avoidance
block pruning
local reorder (formal stage)
```

第一版预实验限制：

> **只做 order-preserving deletion。**

也就是删动作但不改变剩余动作顺序。

正式版再加入：

> 局部 reorder / repeated-pass instance 处理 / nested pipeline 局部调整。

最终目标优先级：

```text
1. Runtime improvement
2. Correctness
3. Execution budget
4. Plan compactness
5. Compile time / code size
```

---

# 6. PatchCredit 和普通 Compiler Phase Ordering 的区别

普通 phase ordering：

```text
从零搜索
→ 找一个好的 pass sequence
```

PatchCredit：

```text
LLM 已经给了一个 Plan
→ 分析这个 Plan 内部为什么有效/无效
→ 在该 Plan 邻域进行低预算修正
```

因此研究问题是：

> **Post-hoc diagnosis and repair of LLM-generated compiler plans**

而不是：

> “我们又提出一个从零搜索 Pass Sequence 的算法。”

这个边界必须贯穿全文，否则很容易被 Compiler-R1、ECCO、AutoPass、传统 phase-ordering 工作覆盖。

---

# 7. 数据到底从哪里来

PatchCredit 不要求自己发明新的 benchmark。

但必须区分：

## 7.1 Benchmark Dataset

这是给 LLM Compiler Agent 优化的原程序，例如：

- CompilerGym `cbench-v1`
- PolyBench/C
- LLVM test-suite

## 7.2 LLM Plan Dataset

这是我们真正要构建的数据：

```text
Benchmark
→ LLM Compiler Agent
→ Plan
→ compile
→ correctness
→ repeated runtime
→ accepted / rejected
```

## 7.3 PatchCredit Dataset

只有同时满足：

```text
compile PASS
correctness PASS
runtime improvement > noise threshold
plan length within selected range
plan replayable
```

的 LLM Plans，才进入 PatchCredit 主实验。

因此：

> **现成 benchmark 有；现成 PatchCredit 输入集目前不假设存在，必须生成。**

---

# 8. 预实验首选数据方案

## 第一候选：CompilerGym cbench-v1

官方 CompilerGym LLVM 文档列出：

```text
benchmark://cbench-v1
23 runnable C benchmarks
Validatable: Partially
```

同时 LLVM 环境暴露 Runtime observation，可以编译并执行 benchmark，返回多次 wall-clock times。

但官方明确把 Runtime API 标成 experimental，因此预实验必须：

- 保存 raw times；
- 自己计算统计量；
- 检查每个 benchmark 是否真的 runnable；
- 不把 CompilerGym Runtime 单接口当作唯一真值；
- 对部分无 validator / validator 被禁用的任务自己补 correctness policy。

官方来源：
- https://compilergym.com/llvm/index.html

## 第二候选：PolyBench/C

PolyBench/C 是 30 个数值计算 kernel 的集合，强调统一 execution / monitoring。

优点：

- 小；
- 编译快；
- 运行路径稳定；
- 特别适合 4–8 action plan 的穷举 Oracle；
- 可作为 cBench 的第二独立验证集。

来源：
- https://www.cs.colostate.edu/~pouchet/software/polybench/

## 正式扩展：LLVM test-suite

LLVM 官方 test-suite：

- 有 benchmark/test programs；
- 有 reference outputs 用于 correctness；
- 工具支持 runtime、compile time、code size。

适合正式论文做更广任务类型扩展。

来源：
- https://llvm.org/docs/TestSuiteGuide.html

---

# 9. 为什么不能直接拿 Compiler-R1 数据当 Runtime Ground Truth

Compiler-R1 非常有用：

- LLM + RL；
- compiler pass sequence auto-tuning；
- 有公开代码/数据；
- 可以参考 Plan 生成和环境交互。

但其公开核心目标主要是：

```text
reduce LLVM IR instruction count
```

PatchCredit 的核心 utility 是：

```text
real runtime
```

所以合理用法：

```text
复用 Compiler-R1 的 Agent / prompt / sequence-generation ideas
+
重新在自己的 Runtime Harness 上执行并筛选
```

不能写成：

```text
IR instruction count reduced
=> runtime definitely improved
```

来源：
- https://github.com/Mind4Compiler/Compiler-R1

---

# 10. 当前强竞品和安全边界

## Compiler-R1

已做：
- LLM + RL pass auto-tuning；
- pass synergy graph。

PatchCredit 禁止声称：
- 首次 LLM pass sequence；
- 首次 pass synergy。

## ECCO

已做：
- performance evidence；
- causal reasoning；
- LLM strategist；
- GA compiler search。

PatchCredit 禁止声称：
- 首次 causal/evidence compiler optimization。

## AutoPass

已做：
- compiler/runtime evidence；
- LLM agents；
- iterative tuning。

PatchCredit 不能退化为：

```text
runtime bad
→ tell LLM
→ LLM retries
```

必须保留显式 attribution layer。

## Per-Pass LLVM Study

已系统研究固定 `-O3` pipeline 的 cumulative prefixes 和 per-pass performance effects。

PatchCredit 禁止声称：
- 首次 individual pass runtime contribution。

安全边界：

> **arbitrary LLM-generated plan + action-instance + contextual counterfactual runtime attribution + budgeted interventions + recomposition**

---

# 11. 研究问题 RQ

## RQ1
LLM-generated compiler plans 中，负贡献、冗余和 context-dependent interaction 是否是稳定现象？

## RQ2
Contextual Credit 是否优于 Prefix / LOO 来恢复 action importance？

## RQ3
在固定 execution budget 下，PatchCredit sampler 是否比简单采样更有效？

## RQ4
PatchCredit recomposition 是否能真实超过原始 LLM Plan 的 Runtime？

## RQ5
效果是否随着 plan length 增长仍然成立？

## RQ6
IR instruction count 与真实 runtime 不一致时，PatchCredit 是否能够识别这种代理指标失效？

---

# 12. 最小实现边界

第一版必须控制范围：

```text
LLVM only
linear / serializable pass plan
real runtime
4-8 actions for exact prestudy
10-30+ actions for formal scaling
order-preserving deletion first
no LLM fine-tuning
no RL training
no multi-platform in first pilot
```

后续再扩展：

```text
local reorder
nested pipeline
secondary hardware
PassBench / graph compiler
```

---

# 13. Go / No-Go 原则

PatchCredit 不应该默认一定成立。

最关键的失败条件：

```text
1. LLM 成功 Runtime Plan 很难获得
2. Runtime noise 太大
3. Multi-action interaction 很少
4. Full LLM Plan 基本总是最优
5. LOO 已经足够
6. Budgeted Sampling 没比 Random/LOO 更好
7. Recomposition 经常造成回退
```

遇到 3/4/5 中任意两个严重失败：

> 建议停止当前主框架，而不是继续堆算法。

---

# 14. 最理想论文结果

理想案例：

```text
Baseline:       100 ms
LLM Plan:        82 ms
PatchCredit:     74 ms

LLM Plan length: 18
PC Plan length:  11

Execution budget:
Exhaustive impossible
Random search: 100 evaluations
PatchCredit:    30 evaluations
```

论文故事：

> LLM 已经找到一个不错的编译优化计划，但该计划包含上下文相关的冗余、负贡献和协同动作。PatchCredit 通过真实反事实执行恢复这些动作的性能 Credit，在有限测量预算下重组计划，并进一步提升 Runtime。

---

# 15. 当前推荐执行顺序

```text
先执行：
03_预实验可行性研究框架.md

只做到：
P0 → P5

重点看：
P4 / P5

如果没有核心现象：
停止

如果现象成立：
继续 P6 → P9

只有预实验 GO：
才执行 02_完整实验研究框架.md
```
# 16. 预实验 Baseline 最小化说明（最新版）

预实验阶段不执行正式论文级的大量方法对比。

核心对象固定为：

```text
LLM Plan
LOO
PatchCredit
Exact Oracle
```

其中：

```text
LOO         = 预实验唯一真正的方法 Baseline
LLM Plan    = 研究起点
PatchCredit = 我们的方法
Oracle      = 4–8 action 小计划 Ground Truth / 理论上限
```

`-O3` 仍在数据筛选阶段测量，用于确认：

```text
LLM Plan 是否是真正有效的性能优化
```

但不把 `-O3` 当作 PatchCredit 核心方法 Baseline。

预实验暂不要求实现：

```text
Prefix
Random Search
Shapley
Monte Carlo Shapley
Genetic Search
Compiler-R1 / ECCO / AutoPass 的完整复现
```

这些保留到正式实验。

因此 Pilot 的判断顺序简化为：

```text
1. LLM Plan vs Oracle
   → 有没有可优化空间？

2. LOO vs PatchCredit
   → 简单方法够不够？

3. PatchCredit vs LLM + Oracle
   → 能不能在有限预算下把空间转成真实 Runtime gain？
```
