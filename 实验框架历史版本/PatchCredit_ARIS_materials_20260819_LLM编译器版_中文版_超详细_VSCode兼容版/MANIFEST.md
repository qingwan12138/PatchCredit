# PatchCredit 详细材料包 MANIFEST

> Run ID：20260819_135300  
> 语言：中文  
> Markdown：VS Code 默认预览兼容，不依赖 LaTeX 数学插件  
> 状态：研究设计详细版，尚未运行预实验

| 文件 | 用途 | 本版增强 |
|---|---|---|
| `IDEA_REPORT.md` | 完整研究定义 | 加入严格对象定义、数据层次、创新技术边界、RQ、Go/No-Go |
| `IDEA_REPORT_20260819_135300.md` | 主报告快照 | 与固定文件一致 |
| `COMPETITOR_MATRIX.md` | 竞品与创新边界 | 增加 LOO/Shapley/phase-ordering 算法风险与 safe claims |
| `COMPETITOR_MATRIX_20260819_135300.md` | 竞品快照 | 与固定文件一致 |
| `01_request.md` | Novelty 审查请求 | 明确 A1/A2/A3 和判撞标准 |
| `01_response.md` | Novelty 审查结论 | 明确 Conditional Pass 和强 baseline 风险 |
| `02_完整实验研究框架.md` | 正式实验实施手册 | E0-E12，每阶段代码、Gate、产出 |
| `03_预实验可行性研究框架.md` | 预实验实施手册 | 数据集章节大幅扩充，P0-P9 |
| `MANIFEST.md` | 包说明 | 当前文件映射和执行顺序 |
| `run.meta.json` | 机器可读元信息 | 数据源、创新点、状态 |

---

# 本版最重要的修改

## 1. 数据集问题写死

明确：

```text
Benchmark Dataset != PatchCredit Dataset
```

第一候选：

```text
CompilerGym cbench-v1
23 runnable C benchmarks
Partially validatable
```

因此先审计再使用。

第二候选：

```text
PolyBench/C
30 numerical kernels
```

正式扩展：

```text
LLVM test-suite
```

---

## 2. 主数据必须真实由 LLM 生成

```text
Benchmark
→ LLM Compiler Agent
→ Plan
→ Build
→ Correctness
→ Runtime
→ Filter
→ PatchCredit Dataset
```

专家 Plan / fixed O3 只能校准。

---

## 3. 预实验不再笼统写“10-20 tasks”

现在明确了：

```text
cBench:
10-15 task candidates
3-5 LLM attempts/task

PolyBench:
8-10 kernels
2-3 LLM attempts/kernel

目标：
8-20 accepted 4-8-action LLM plans
```

---

## 4. 正式实验扩展到完整工程手册

加入：

```text
TaskSpec
ActionInstance
RuntimeSample
Plan schema
Intervention engine
Dataset freeze
Exact Oracle
Contextual Credit
Budgeted Sampler
Recomposition
Large-Plan Scaling
Full Evaluation
Reproducibility
```

---

# 推荐下一步

不要立刻实现所有文件。

第一开发批次只做：

```text
P0 环境
P1 cBench/PolyBench audit
P2 baseline/noise
P3 LLM Plan generation
P4 accepted plan dataset
P5 exact oracle
```

完成 P5 后再决定是否值得继续。
