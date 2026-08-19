# PatchCredit 最新材料包 MANIFEST

> Run ID：20260819_145500  
> 语言：中文  
> Markdown：VS Code 默认预览兼容  
> 状态：**轻量预实验 + 3B 本地模型冻结版**

## 本次新增决策

预实验默认本地模型正式冻结为：

```text
Qwen2.5-Coder-3B-Instruct
```

硬件：

```text
RTX 4090 24GB
```

模型角色：

```text
只负责生成初始 LLVM 优化 Plan
```

真正的性能判断仍由：

```text
LLVM Build
Correctness
Repeated Runtime
PatchCredit
```

完成。

---

# 文件状态

| 文件 | 本次状态 |
|---|---|
| `IDEA_REPORT.md` | 更新：加入 3B 模型冻结、职责、升级规则 |
| `IDEA_REPORT_20260819_145500.md` | 最新快照 |
| `COMPETITOR_MATRIX.md` | 内容不改 |
| `COMPETITOR_MATRIX_20260819_145500.md` | 新快照 |
| `01_request.md` | 内容不改 |
| `01_response.md` | 内容不改 |
| `02_完整实验研究框架.md` | 小改：加入正式实验模型策略 |
| `03_预实验可行性研究框架.md` | 更新：P3 固定 Qwen2.5-Coder-3B-Instruct |
| `MANIFEST.md` | 更新 |
| `run.meta.json` | 更新 |

---

# 当前预实验最小方法集

```text
LLM Plan
LOO
PatchCredit
Exact Oracle
```

其中：

```text
LOO = 唯一真正 Pilot Baseline
```

---

# 当前预实验数据链

```text
cBench / PolyBench
↓
Qwen2.5-Coder-3B-Instruct
↓
4–8 action LLVM Plan
↓
Build
↓
Correctness
↓
Repeated Runtime
↓
Accepted LLM Plan Dataset
↓
Exact Oracle
↓
LOO vs PatchCredit
↓
Go / No-Go
```

---

# 模型升级规则

只有当 3B 明显成为数据瓶颈时才允许升级。

顺序：

```text
3B
→ 7B
→ 更大模型
```

任何模型升级都必须新建独立数据版本，不能覆盖 3B 数据。

---

# 推荐下一步

先做：

```text
P0 Runtime 环境校准
P1 Benchmark 审计
P2 Baseline / 噪声
```

在 P0–P2 完成前，暂时不需要启动 3B 模型生成 Plan。

之后进入：

```text
P3 = Qwen2.5-Coder-3B-Instruct 生成真实 Plan
```
