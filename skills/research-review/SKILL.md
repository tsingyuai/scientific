---
name: research-review
description: "Review ML implementation against plan and survey. Iterates fix-rerun-review up to 3 times. Requires ml_res.md from /research-implement."
metadata:
  {
    "openclaw":
      {
        "emoji": "🔍",
        "requires": { "bins": ["python3", "uv"] },
      },
  }
---

# Research Review

**Don't ask permission. Just do it.**

**Workspace:** See `../_shared/workspace-spec.md`. Set `$W` to the active project directory.

## Prerequisites

| File | Source |
|------|--------|
| `$W/ml_res.md` | /research-implement |
| `$W/project/` | /research-implement |
| `$W/plan_res.md` | /research-plan |
| `$W/survey_res.md` | /research-survey |

**If `ml_res.md` is missing, STOP:** "需要先运行 /research-implement 完成代码实现"

## Output

| File | Content |
|------|---------|
| `$W/iterations/judge_v{N}.md` | 每轮审查报告 |

最终报告中 `verdict: PASS` 表示审查通过。

---

## Workflow

### Step 1: 审查代码

读取以下内容：
- `$W/plan_res.md` — 每个组件的预期
- `$W/survey_res.md` — 核心公式
- `$W/project/` — 实际代码
- `$W/ml_res.md` — 执行结果

### Step 2: 逐项检查

| 检查项 | 方法 |
|--------|------|
| 数据管道匹配 plan | 对比 plan Dataset Plan vs `data/` 实现 |
| 模型架构匹配公式 | 对比 survey 公式 vs `model/` 实现 |
| Loss 函数正确 | 对比 plan Training Plan vs `training/loss.py` |
| 评估指标正确 | 对比 plan Testing Plan vs `testing/` |
| [RESULT] 行存在 | 检查 ml_res.md 中的数值来源 |
| Loss 合理 | 非 NaN/Inf，有下降趋势 |
| 无 mock 数据（除非已声明） | 搜索 `# MOCK DATA` 注释 |

### Step 3: 写入审查报告

写入 `$W/iterations/judge_v1.md`：

```markdown
# Review v1

## Verdict: PASS / NEEDS_REVISION

## Checklist
- [x/✗] Dataset loading matches plan
- [x/✗] Model architecture matches formulas
- [x/✗] Loss function correct
- [x/✗] Training loop proper
- [x/✗] Evaluation metrics correct
- [x/✗] Results are from real execution (not fabricated)

## Issues (if NEEDS_REVISION)
1. **{issue}**: {description} → **Fix**: {specific fix instruction}
2. ...
```

### Step 4: 迭代（如果 NEEDS_REVISION）

循环最多 3 次：

1. 读取 `judge_v{N}.md` 的修改建议
2. 修改 `$W/project/` 中的代码
3. 重新执行：
   ```bash
   cd $W/project && source .venv/bin/activate && python run.py --epochs 2
   ```
4. 读取执行输出，验证修复
5. 写入 `judge_v{N+1}.md`
6. 如果 PASS → 停止；否则继续

### Step 5: 最终判定

3 轮后仍 NEEDS_REVISION → 在最后一份 judge 中列出剩余问题，标记 `verdict: BLOCKED`，等待用户介入。

---

## Rules

1. 审查必须逐项对照 plan，不能只看"代码能跑"
2. 每个 issue 必须给出具体的修复指令（不是"请改进"）
3. 验证修复后必须重新执行代码并检查输出
4. PASS 的前提：所有 checklist 项通过 + [RESULT] 数值合理
