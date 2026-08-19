# ARIS 输出清单：PatchCredit（LLM 编译器版）

> 版本：2026-08-19 / run `20260819_132900`

| 文件 | 固定路径 | 版本快照 | 状态 |
|---|---|---|---|
| 新版主研究框架 | `IDEA_REPORT.md` | `IDEA_REPORT_20260819_132900.md` | 已收紧为 LLM Compiler Agent optimization plan |
| 新版竞品矩阵 | `COMPETITOR_MATRIX.md` | `COMPETITOR_MATRIX_20260819_132900.md` | 已加入 Compiler-R1 / ECCO / AutoPass / Per-Pass Study 等强近邻边界 |
| 创新性复查请求 | `01_request.md` | — | 已按 contextual credit 新主张重写 |
| 创新性复查结论 | `01_response.md` | — | 当前无完整直接撞车，但竞品压力高 |
| 完整正式实验框架 | `02_完整实验研究框架.md` | — | E0–E10，含代码指导、Gate、产出清单 |
| 预实验框架 | `03_预实验可行性研究框架.md` | — | P0–P8，主数据改为真实 LLM compiler plans |
| 元数据 | `run.meta.json` | — | 本次版本信息 |

## 本次相对上一版的核心修改

1. 研究对象从 **LLM source-code multi-edit Patch** 改为 **LLM Compiler Agent 生成的多动作编译优化计划**。
2. 主创新从泛化的 performance credit 改为 **Contextual Counterfactual Credit Assignment**。
3. “编辑”统一改为 **action instance**，与 pass 名称、位置、参数和 pipeline 上下文绑定。
4. 预实验不再用专家 Patch 证明有效性，专家/固定 O3 只用于工具校准。
5. 主数据链固定为：`Benchmark → LLM Compiler Agent → successful plan → PatchCredit`。
6. 预实验 4–8 actions 仅用于 exact oracle；正式实验支持 10–30+ actions。
7. 正式加入 hierarchical/budgeted intervention selection 处理组合爆炸。
8. 重组第一版限制为 order-preserving pruning，后续再增加 local reorder。
9. 竞品边界重新锁定：不能声称首次 LLM pass tuning、首次 pass synergy、首次 causal evidence、首次 per-pass contribution 或首次 runtime-feedback agent。
10. 最终成功标准仍以 **真实 runtime 进一步提升** 为第一优先级。

## 当前推荐执行顺序

```text
先跑 03_预实验可行性研究框架.md
P0 → P4
```

P4 是第一道生死 Gate。

如果真实 LLM-generated plans 中根本没有稳定 interaction / negative action / better subplan，停止 PatchCredit。

如果 P4 通过，再做 P5–P7。

只有 P7 通过，才进入 `02_完整实验研究框架.md`。
