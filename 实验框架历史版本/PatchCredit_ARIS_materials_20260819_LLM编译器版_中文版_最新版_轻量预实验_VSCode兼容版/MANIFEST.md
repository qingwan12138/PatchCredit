# PatchCredit 最新材料包 MANIFEST

> Run ID：20260819_142100  
> 语言：中文  
> Markdown：VS Code 默认预览兼容，不依赖 LaTeX 插件  
> 状态：**最新版轻量化预实验设计；尚未运行 Pilot**

| 文件 | 状态 | 本次是否实质修改 |
|---|---|---|
| `IDEA_REPORT.md` | 最新 | 小改：增加预实验 Baseline 最小化说明 |
| `IDEA_REPORT_20260819_142100.md` | 最新快照 | 同主报告 |
| `COMPETITOR_MATRIX.md` | 保留上一详细版 | 否 |
| `COMPETITOR_MATRIX_20260819_142100.md` | 新快照 | 内容未改 |
| `01_request.md` | 保留 | 否 |
| `01_response.md` | 保留 | 否 |
| `02_完整实验研究框架.md` | 基本保留 | 只增加“正式 Baseline 不要求 Pilot 提前实现”说明 |
| `03_预实验可行性研究框架.md` | **重点重写** | 是 |
| `MANIFEST.md` | 最新 | 是 |
| `run.meta.json` | 最新 | 是 |

---

# 本次核心修改

之前预实验阶段列出的 Baseline 偏多。

当前正式收缩为：

```text
LLM Plan
→ 研究起点

LOO
→ Pilot 唯一真正的简单 Baseline

PatchCredit
→ 我们的方法

Exact Oracle
→ 4–8 action 小计划 Ground Truth
```

`-O3` 仍然要测，但它只用于：

```text
确认 LLM Plan 确实是有效性能优化
```

不作为 PatchCredit Pilot 核心方法 Baseline。

---

# 最新 Pilot 判断顺序

```text
P5:
LLM Plan vs Oracle
→ 是否存在可优化空间？

P6:
LOO vs PatchCredit
→ 简单方法是否已经足够？

P7:
PatchCredit 在有限测量预算下在线重组
→ 是否能真实超过 LLM Plan，并接近 Oracle？
```

---

# 暂时移出预实验的内容

以下只保留在正式实验：

```text
Prefix
Random Search
Shapley
Monte Carlo Shapley
Genetic / Local Search
完整强系统复现
```

这样做的目的：

> **先验证 Idea 是否成立，再投入论文级完整评测工程。**

---

# 推荐下一步

严格按：

```text
03_预实验可行性研究框架.md
P0 → P5
```

先完成。

尤其 P5：

```text
LLM Plan vs Exact Oracle
```

如果大部分 LLM Plan 已经接近 Oracle，则不用继续开发复杂 PatchCredit。
