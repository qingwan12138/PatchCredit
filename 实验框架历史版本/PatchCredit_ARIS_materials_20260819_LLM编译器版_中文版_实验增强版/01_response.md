# 01_response：PatchCredit（LLM 编译器版）创新性复查结果

## 结论

**当前未发现一个强竞品完整覆盖 A1+A2+A3 的整条链，但单项邻域已经非常拥挤。**

因此结论不是“无竞品”，而是：

> **可以继续，但必须把新颖性锁在“LLM-generated plan 的 contextual counterfactual runtime credit + budgeted estimation + recomposition”组合上。**

## 主要碰撞

### Compiler-R1

已经做 LLM+RL compiler pass auto-tuning，并包含 pass synergy graph。

结论：
- “LLM 生成 pass sequence”不是新；
- “pass synergy”不是新；
- PatchCredit 必须是 post-hoc plan attribution。

参考：
- https://arxiv.org/abs/2506.15701
- https://github.com/Mind4Compiler/Compiler-R1

### ECCO

已经做 evidence-driven causal reasoning for compiler optimization，LLM strategist 指导 GA。

结论：
- “因果 / evidence compiler optimization”不能当新颖性；
- PatchCredit 的 attribution 必须发生在 plan 已经生成以后。

参考：
- https://arxiv.org/abs/2602.00087

### Synergy-Guided Compiler Auto-Tuning

已经 mining pass synergy 并用于 LLVM NPM search。

结论：
- pairwise synergy 不能单独成为创新；
- PatchCredit 要求 plan-conditioned action-instance interaction。

参考：
- https://arxiv.org/abs/2510.13184

### Per-Pass LLVM Study

已经系统研究固定 `-O3` pipeline 的 cumulative per-pass runtime impact 和 phase interference。

结论：
- individual pass contribution 不是新；
- PatchCredit 必须突出 arbitrary LLM plan + non-prefix intervention + contextuality。

参考：
- https://arxiv.org/abs/2606.31238

### AutoPass

已经使用 compiler/runtime evidence 反馈给 LLM agent 做迭代性能 tuning。

结论：
- runtime feedback → LLM retry 不是新；
- PatchCredit 必须有显式、独立、可评价的 credit model。

参考：
- https://arxiv.org/abs/2606.20373

### TRIM

已经做 agent patch 的 counterfactual deletion/minimization。

结论：
- 删除/最小化不能作为主创新。

参考：
- https://arxiv.org/abs/2607.18161

## 当前可守边界

PatchCredit 当前最有价值的组合边界是：

> 对一个已经由 LLM Compiler Agent 生成的具体优化计划，基于真实 runtime 的可执行反事实干预，在 action-instance 级估计上下文性能贡献和交互；在无法穷举组合时做预算化干预选择；最后用归因结果进行二次重组，并实测是否超过原始 LLM plan。

## 当前内部评级

- 研究问题清晰度：8.5/10
- LLM Compiler 相关性：9.0/10
- 单点新颖性：6.5/10
- 组合新颖性：7.8/10
- 可实现性：8.0/10
- 竞品压力：高
- 是否值得预实验：**是**

注意：以上是研究设计内部评价，不代表录用概率。

## 下一步

不继续扩大概念，而是立刻做 Go/No-Go Pilot：

1. 生成真实 LLM compiler plans；
2. 筛 4–8 actions 的成功 plan；
3. 穷举 order-preserving deletion subsets；
4. 检查 contextual interaction 与更优子计划是否真实存在；
5. 再决定是否实现复杂 budgeted estimator。
