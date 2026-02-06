---
name: research-pipeline
description: "Orchestrates the full research workflow by spawning sub-agents for each phase. Checks workspace state, dispatches tasks, verifies outputs. Use for: end-to-end ML research. Each phase runs in an isolated context via sessions_spawn."
metadata:
  {
    "openclaw":
      {
        "emoji": "🔬",
        "requires": { "bins": ["git", "python3", "uv"] },
      },
  }
---

# Research Pipeline (Orchestrator)

**Don't ask permission. Just do it.**

你是编排器。你不直接做研究工作，而是：
1. 检查 workspace 文件状态
2. 为下一步构造任务描述
3. 用 `sessions_spawn` 派发给子 agent
4. 等待完成后验证产出
5. 重复直到流程结束

**Workspace:** See `../_shared/workspace-spec.md`. Set `$W` to the active project directory.

---

## Step 0: 初始化

```bash
ACTIVE=$(cat ~/.openclaw/workspace/projects/.active 2>/dev/null)
```

如果没有 active project：
1. 问用户：研究主题是什么？
2. 创建项目目录
3. 写入 `task.json`

设置 `$W = ~/.openclaw/workspace/projects/{project-id}`

---

## 调度循环

按顺序检查每个阶段。**每次只执行一个阶段。**

### Phase 1: Literature Survey

**检查:** `$W/papers/_meta/` 目录存在且有 `.json` 文件？

**如果缺失，spawn:**

```
sessions_spawn({
  task: "工作目录: $W\n执行 /literature-survey 技能\n\n研究主题: {从 task.json 提取}\n请搜索、筛选、下载相关论文到 $W/papers/",
  label: "Literature Survey"
})
```

**验证:** `ls $W/papers/_meta/*.json` 至少有 3 个文件

---

### Phase 2: Deep Survey

**检查:** `$W/survey_res.md` 存在？

**如果缺失，先读取 Phase 1 摘要，然后 spawn:**

```
sessions_spawn({
  task: "工作目录: $W\n执行 /research-survey 技能\n\n上下文: 已下载 {N} 篇论文，方向包括 {directions}\n请深度分析论文，提取公式，写入 survey_res.md",
  label: "Deep Survey"
})
```

**验证:** `$W/survey_res.md` 存在且包含"核心方法对比"表格

---

### Phase 3: Implementation Plan

**检查:** `$W/plan_res.md` 存在？

**如果缺失，读取 survey_res.md 摘要，然后 spawn:**

```
sessions_spawn({
  task: "工作目录: $W\n执行 /research-plan 技能\n\n上下文: 调研发现核心方法是 {method}，推荐技术路线 {route}\n请制定完整实现计划到 plan_res.md",
  label: "Research Plan"
})
```

**验证:** `$W/plan_res.md` 存在且包含 4 个 section（Dataset/Model/Training/Testing）

---

### Phase 4: Implementation

**检查:** `$W/ml_res.md` 存在？

**如果缺失，读取 plan_res.md 要点，然后 spawn:**

```
sessions_spawn({
  task: "工作目录: $W\n执行 /research-implement 技能\n\n上下文:\n- 计划包含 {N} 个组件: {list}\n- 数据集: {dataset}\n- 框架: PyTorch\n请实现代码到 $W/project/，运行 2 epoch 验证，写入 ml_res.md",
  label: "Research Implement"
})
```

**验证:**
- `$W/project/run.py` 存在
- `$W/ml_res.md` 包含 `[RESULT]` 行
- loss 值非 NaN/Inf

---

### Phase 5: Review

**检查:** `$W/iterations/` 下最新 `judge_v*.md` 的 verdict 是否为 PASS？

**如果没有 PASS，spawn:**

```
sessions_spawn({
  task: "工作目录: $W\n执行 /research-review 技能\n\n上下文:\n- 实现报告: ml_res.md 显示 train_loss={value}\n- 计划在 plan_res.md\n请审查代码，如需修改则迭代修复（最多 3 轮）",
  label: "Research Review"
})
```

**验证:** 最新 `judge_v*.md` 中 `verdict: PASS` 或 `verdict: BLOCKED`

如果 BLOCKED → 报告用户，等待指示

---

### Phase 6: Full Experiment

**检查:** `$W/experiment_res.md` 存在？

**如果缺失，spawn:**

```
sessions_spawn({
  task: "工作目录: $W\n执行 /research-experiment 技能\n\n上下文:\n- Review PASS，代码已验证\n- plan_res.md 中指定 full epochs\n请执行完整训练 + 消融实验，写入 experiment_res.md",
  label: "Research Experiment"
})
```

**验证:** `$W/experiment_res.md` 包含 `[RESULT]` 行和消融表格

---

## 完成

所有 Phase 验证通过后，输出最终摘要：

```
研究流程完成！
- 论文: {N} 篇分析
- 代码: $W/project/
- 结果: $W/experiment_res.md
- 审查: $W/iterations/ ({N} 轮)
```

---

## 上下文桥接规则

每次 spawn 前，编排器必须：
1. **读取**上一步的产出文件
2. **摘要** 2-5 行关键信息（不要复制全文）
3. **写入** spawn task 的"上下文"部分

这确保子 agent 拿到足够信息启动，同时不会被前序步骤的完整输出污染。

## Recovery

如果编排器中断：
1. 重新运行 /research-pipeline
2. 编排器会自动检查所有文件，跳过已完成的阶段
3. 从第一个缺失的产出文件开始继续
