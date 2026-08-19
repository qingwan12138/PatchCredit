# ARIS 输出清单

## PatchCredit 修订版——2026-08-19 / run `20260819_123200`

| 文件 | 固定路径 | 版本快照路径 | 状态 |
|---|---|---|---|
| PatchCredit 修订版研究框架 | `IDEA_REPORT.md` | `IDEA_REPORT_20260819_123200.md` | 主创新已改为“性能贡献 + 编辑交互”；尚未运行 Pilot |
| 修订后的竞品/碰撞矩阵 | `COMPETITOR_MATRIX.md` | `COMPETITOR_MATRIX_20260819_123200.md` | 已加入 TRIM；Patch 最小化降级；Motif Distiller 降为扩展 |
| 创新性复查请求 | `01_request.md` | — | 修订后的 A1–A4 主张 |
| 创新性复查结果 | `01_response.md` | — | PatchCredit 修订版：存在部分重叠，但组合边界仍可辩护 |
| 运行元数据 | `run.meta.json` | — | 已完成文献筛查；没有声称任何已完成实验结果 |

## 相比 2026-08-18 压缩包的主要变化

1. 新增 **TRIM**，作为 Agent Patch 最小化方向最强直接近邻之一。
2. Patch 最小化不再作为正式创新点。
3. 主创新改为 **依赖感知的性能贡献与交互建模（Dependency-aware Performance Credit & Interaction Modeling）**。
4. 支撑创新 1 改为 **预算化交互采样（Budgeted Interaction Sampling）**。
5. 支撑创新 2 改为 **性能引导的 Patch 重组（Performance-guided Patch Recomposition）**。
6. 优化目标优先追求达到或超过完整 LLM Patch 的运行时性能；Patch 大小仅作为次要正则项/Pareto 目标。
7. `Motif Distiller` 降级为可选扩展，因为策略提取与复用方向已有较多工作（例如 SemOpt）。
8. CostWitness 继续只作为条件备选，不建议与 PatchCredit 并行实现。

## 推荐的下一步 Gate

先运行一个小规模穷举 Pilot：选择 10–15 个正确、显著加速、包含 4–8 个依赖闭合编辑单元的 LLM 多编辑 Patch。先验证非加性交互和 Patch 重组机会是否真实存在，再实现复杂的预算化采样器。

本压缩包**尚未执行任何 PatchCredit 实验**。

## 2026-08-19 新增实验材料

- `02_完整实验研究框架.md`：正式论文实验与工程实施框架，按 E0–E10 严格分阶段，包含代码模块、接口、Gate 与产出清单。
- `03_预实验可行性研究框架.md`：Go/No-Go 预实验框架，按 P0–P7 分阶段，优先用 GSO 小规模穷举验证核心现象，再决定是否进入完整实现。
