---
name: research-experiment
description: "[Read when prompt contains /research-experiment]"
metadata:
  {
    "openclaw":
      {
        "emoji": "🧪",
        "requires": { "bins": ["python3", "uv"] },
      },
  }
---

# Research Experiment

**Don't ask permission. Just do it.**

**Workspace:** See `../_shared/workspace-spec.md`. Set `$W` to the active project directory.

## Prerequisites

| File | Source |
|------|--------|
| `$W/project/` | /research-implement |
| `$W/plan_res.md` | /research-plan |
| `$W/iterations/judge_v*.md` | /research-review（最后一份 verdict 必须是 PASS） |

**验证 PASS：** 读取最新的 `judge_v*.md`，确认 `verdict: PASS`。如果不是，STOP。

## Output

| File | Content |
|------|---------|
| `$W/experiment_res.md` | 完整实验报告（含 full training + 消融 + 补充实验） |
| `$W/experiment_analysis/analysis_{N}.md` | 每轮实验分析报告（迭代过程中产生） |

---

## Workflow

### Step 1: Full Training

修改 epoch 数为 plan_res.md 中指定的正式值。**不要改代码逻辑，只改 epoch。**

```bash
cd $W/project && source .venv/bin/activate
python run.py  # full epochs
```

记录完整训练的 `[RESULT]` 输出。

### Step 2: 分析结果

读取训练输出，评估：
- 最终 loss 和 metrics
- 训练曲线趋势（loss 是否持续下降）
- 是否过拟合（train vs val gap）

### Step 3: 消融实验

根据 plan_res.md 中的消融计划，执行 2-3 个消融实验：

对每个消融：
1. 修改代码（注释/替换对应组件）
2. 执行 2 epoch 快速验证
3. 记录结果

```bash
# Example: 去掉 attention module
python run.py --epochs 2 --ablation no_attention
```

### Step 4: 实验分析→补充实验迭代（2 轮）

**⚠️ 这是 Novix Exp Analyzer 机制 — 对已有结果进行分析，提出补充实验，执行后再分析。**

循环 **2 次**：

#### 4.1 分析当前结果

读取当前所有实验结果（full training + 消融），写入分析报告 `$W/experiment_analysis/analysis_{N}.md`：

```markdown
# Experiment Analysis Round {N}

## 当前结果摘要
- Full training: {metrics}
- 消融实验: {key findings}

## 发现的问题或机会
1. {observation} → 建议: {experiment}
2. ...

## 补充实验计划
| 实验名称 | 目的 | 修改内容 | 预期结果 |
|----------|------|----------|----------|
| {exp_name} | {why} | {what to change} | {expected} |
```

补充实验类型参考（**Novix Exp Analyzer** 的典型输出）：
- **敏感性分析**：关键超参数（lr、hidden_dim、dropout）的影响
- **可视化**：attention map、embedding 可视化、训练曲线对比图
- **对比实验**：与 baseline 方法的性能对比
- **鲁棒性测试**：不同数据规模/噪声水平下的表现

#### 4.2 执行补充实验

根据分析报告中的计划，修改代码并执行补充实验。**只改实验相关参数/配置，不改核心算法逻辑。**

```bash
cd $W/project && source .venv/bin/activate
python run.py --experiment {exp_name}
```

记录结果后，回到 4.1 进行下一轮分析（共 2 轮）。

---

### Step 5: 写入最终实验报告

汇总所有实验结果（full training + 消融 + 2 轮补充实验），写入 `$W/experiment_res.md`：

```markdown
# Experiment Report

## Full Training Results (from execution log)
- Epochs: {N}
- [RESULT] train_loss={value}
- [RESULT] val_metric={value}
- [RESULT] elapsed={value}
- [RESULT] device={device}

> 以上数值来自真实执行输出。

## Training Analysis
- 收敛情况: {converged / still improving / diverged}
- 过拟合: {yes/no, evidence}

## Ablation Studies

| 实验 | 修改 | val_metric | vs Full |
|------|------|-----------|---------|
| Full model | — | {value} | baseline |
| No {component} | 去掉 {X} | {value} | {-/+}% |
| ... | ... | ... | ... |

## Supplementary Experiments

### Sensitivity Analysis
| 超参数 | 值 | val_metric | 备注 |
|--------|-----|-----------|------|
| ... | ... | ... | ... |

### Comparison with Baselines
| 方法 | val_metric | 备注 |
|------|-----------|------|
| Ours | {value} | — |
| {Baseline} | {value} | ... |

### Visualizations
- 训练曲线: `$W/project/figures/training_curve.png`
- {其他可视化}: `$W/project/figures/{name}.png`

## Conclusions
- {key findings from all experiments}

## Limitations
- {limitations and future work}
```

---

## Rules

1. Full training 只改 epoch 数，不改代码逻辑
2. 所有数值必须来自真实执行输出
3. 消融实验至少做 2 个
4. 如果 full training 失败（OOM 等），调整 batch_size 后重试，不要跳过
5. **补充实验迭代必须做 2 轮（Novix Exp Analyzer 机制）** — 第 1 轮针对初始结果，第 2 轮针对补充实验结果
6. 补充实验不改核心算法，只改实验配置/参数/可视化代码
