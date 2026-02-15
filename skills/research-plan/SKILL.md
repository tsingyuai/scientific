---
name: research-plan
description: "[Read when prompt contains /research-plan]"
metadata:
  {
    "openclaw":
      {
        "emoji": "📋",
      },
  }
---

# Research Plan

**Don't ask permission. Just do it.**

**Workspace:** See `../_shared/workspace-spec.md`. Set `$W` to the active project directory.

## Prerequisites

| File | Source |
|------|--------|
| `$W/task.json` | /research-pipeline or user |
| `$W/survey_res.md` | /research-survey |
| `$W/notes/paper_*.md` | /research-survey |
| `$W/repos/` | /literature-survey Phase 3 |
| `$W/prepare_res.md` | /literature-survey Phase 3 |

**If `survey_res.md` is missing, STOP:** "需要先运行 /research-survey 完成深度分析"

## Output

| File | Content |
|------|---------|
| `$W/plan_res.md` | 四部分实现计划 |

---

## Workflow

### Step 1: 读取上下文

读取以下文件，理解研究目标和技术方案：
- `$W/task.json` — 研究目标
- `$W/survey_res.md` — 技术路线建议、核心公式、**公式→代码映射表**、参考代码架构摘要
- `$W/prepare_res.md` — 参考仓库列表及关键文件说明

### Step 2: 参考代码深度分析

**⚠️ 强制性步骤（Novix Plan Agent 机制）** — 读参考仓库的实现细节，确保 plan 有具体可行的依据。

对 `prepare_res.md` 中的重点仓库：
1. 读取目录结构和 README
2. 读取核心模型代码，理解架构实现方式
3. 读取训练脚本，理解超参数选择和训练技巧
4. 读取数据加载代码，理解预处理流程

为每个组件记录：**参考文件路径 + 关键实现细节**。这些信息将直接填入 plan 的"参考代码"列。

### Step 3: 制定四部分计划

写入 `$W/plan_res.md`：

```markdown
# Implementation Plan

## 1. Dataset Plan

- **数据集名称:** {name}
- **来源:** {URL or description}
- **大小:** {samples / size}
- **预处理步骤:**
  1. {step}
  2. {step}
- **DataLoader 设计:**
  - batch_size: {value}
  - 输入格式: {shape}
  - 输出格式: {shape}

## 2. Model Plan

- **架构概述:** {1-2 sentences}
- **组件列表:**

| 组件 | 对应公式 | 参考代码 | 输入 → 输出 |
|------|----------|----------|-------------|
| {component} | $formula$ | `repos/xxx/file.py` | {shape} → {shape} |

- **参数量估计:** {approximate}

## 3. Training Plan

- **Loss 函数:** {formula + description}
- **Optimizer:** {Adam/SGD/...}, lr={value}
- **Scheduler:** {if any}
- **训练参数:**
  - epochs (validation): 2
  - epochs (full): {value}
  - batch_size: {value}
- **监控指标:** {loss, metrics to log}

## 4. Testing Plan

- **评估指标:**

| Metric | 公式/描述 | 期望范围 |
|--------|-----------|----------|
| {metric} | {description} | {range} |

- **Baselines:** {what to compare against}
- **消融实验（初步规划）:**
  1. {ablation 1}
  2. {ablation 2}
```

### Step 4: 自检

验证计划的完整性：
- [ ] 每个模型组件都有对应公式
- [ ] **每个组件的"参考代码"列已填写**（当 repos/ 存在时）
- [ ] 数据集有具体获取方式（URL 或下载命令）
- [ ] Loss 函数有数学定义
- [ ] 评估指标有明确定义
- [ ] 训练参数合理（不要 lr=0.1 for Adam）

如有不确定项，在计划中标注 `⚠️ TODO: {reason}`

---

## Rules

1. 计划中每个组件必须可追溯到 survey_res.md 中的公式或方法
2. 不要写"通用"计划 — 每个参数都要有具体值或合理估计
3. 如果参考仓库存在，组件表必须包含参考代码路径
4. plan_res.md 的完成标志：四个部分都存在且非空
