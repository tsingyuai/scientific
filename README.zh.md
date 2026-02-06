# Scientify

**为 OpenClaw 打造的 AI 驱动研究工作流自动化插件。**

[English](./README.md)

---

## 功能

### Skills (通过 LLM)

| Skill | 描述 |
|-------|------|
| **research-pipeline** | 端到端 ML 研究编排器。通过 sessions_spawn 逐阶段派发子 agent，验证产出后推进。 |
| **research-survey** | 深度分析已下载论文：提取公式、映射代码、生成核心方法对比表。 |
| **research-plan** | 从调研结果制定四部分实现计划（数据集/模型/训练/测试）。 |
| **research-implement** | 按计划实现 ML 代码，使用 `uv` 虚拟环境隔离，2 epoch 验证，确保真实结果。 |
| **research-review** | 对照计划和调研审查实现代码，最多迭代修复 3 轮。 |
| **research-experiment** | 完整训练 + 消融实验 + 结果分析。需要 review PASS。 |
| **literature-survey** | 文献综述：搜索 → 筛选 → 下载 → 聚类 → 报告。 |
| **idea-generation** | 从研究主题生成创新想法。搜索 arXiv/GitHub、下载论文，输出 5 个研究想法。 |

### Commands (直接执行，不经 LLM)

| 命令 | 描述 |
|------|------|
| `/research-status` | 显示工作空间状态 |
| `/papers` | 列出已下载论文 |
| `/ideas` | 列出已生成想法 |
| `/projects` | 列出所有项目 |
| `/project-switch <id>` | 切换项目 |
| `/project-delete <id>` | 删除项目 |

### Tools

| Tool | 描述 |
|------|------|
| **arxiv_search** | 搜索 arXiv API，返回论文元数据（标题、作者、摘要、ID）。 |
| **arxiv_download** | 按 ID 下载 arXiv 论文，优先 .tex 源文件，回退到 PDF。内置速率限制。 |
| **github_search** | 搜索 GitHub 仓库，支持关键词、语言过滤、按 stars/更新时间排序。 |

---

## 快速开始

```bash
# 安装插件
openclaw plugins install scientify

# 开始使用
openclaw "研究 transformer 效率并生成研究想法"
```

---

## 安装

```bash
openclaw plugins install scientify
```

> **注意：** 不要使用 `npm install scientify`。OpenClaw 插件必须通过 `openclaw plugins install` 安装才能被正确发现。

插件将安装到 `~/.openclaw/extensions/scientify/` 并自动启用。

---

## 使用场景

### 1. 一键生成研究想法

```
你: 帮我调研 "长文档摘要" 领域，生成一些创新的研究想法

Agent: [自动执行]
  1. 搜索 arXiv 论文
  2. 搜索 GitHub 仓库
  3. 下载并分析 .tex 源文件
  4. 生成 5 个创新想法
  5. 选出最佳想法并增强
  6. 映射到代码实现
```

### 2. 每日文献追踪

```
你: 帮我设置一个定时任务，每天早上 9 点检查 arXiv 上关于
   "transformer efficiency" 的新论文，如果有相关的就总结后发到飞书

Agent: 好的，我来设置:
  1. 创建定时 Hook (cron: "0 9 * * *")
  2. 每天调用 arxiv 工具搜索最近 24h 的论文
  3. 用你的 idea (selected_idea.md) 作为对比基准
  4. 筛选相关论文，生成摘要
  5. 通过飞书 webhook 推送

[示例推送]
📚 今日相关论文 (3篇)
• "FlashAttention-3: Fast Attention with ..." - 与你的 idea 高度相关，建议精读
• "Efficient Long-Context Transformers" - 中等相关
• "..."
```

### 3. 研究进展汇报

```
你: 每周五下午帮我生成本周研究进展报告，发到 Slack

Agent: [每周五自动执行]
  1. 读取 $WORKSPACE 变化
  2. 统计: 新增论文 X 篇，新想法 Y 个，代码提交 Z 次
  3. 生成周报 markdown
  4. 推送到 Slack #research 频道
```

### 4. 竞品论文监控

```
你: 监控 "Yann LeCun" 和 "Meta AI" 的新论文，有新发布就提醒我

Agent: 设置监控:
  - 每日检查 arxiv 作者 "Yann LeCun"
  - 检查 arxiv 机构 "Meta AI"
  - 新论文出现时推送通知
```

### 5. 论文阅读助手

```
你: 帮我读一下 papers/2401.12345/ 这篇论文，
   重点关注它的方法和我的 idea 有什么异同

Agent: [读取论文 .tex 文件]

  📄 论文: "Efficient Attention for Long Documents"

  ## 与你的 idea 对比

  | 方面 | 论文方法 | 你的方法 |
  |------|---------|---------|
  | 注意力机制 | Sparse Attention | Hierarchical Attention |
  | 复杂度 | O(n√n) | O(n log n) |
  | 优势 | 实现简单 | 保留更多信息 |

  ## 建议
  可以考虑结合两者...
```

### 6. Related Work 写作助手

```
你: 基于我项目里的论文，帮我写 Related Work 章节

Agent: [读取 papers/ 和 literature/notes/]

  生成 related_work.tex:

  \section{Related Work}

  \subsection{Long Document Summarization}
  Recent works have explored various approaches...
  \cite{paper1} proposed...
  \cite{paper2} extended this by...

  \subsection{Efficient Transformers}
  ...
```

### 7. 想法碰撞

```
你: 把 "强化学习" 和 "文本摘要" 两个领域结合，看能不能碰撞出新想法

Agent: [搜索两个领域的论文]

  🔥 碰撞结果:

  1. **RL-guided Summarization**
     用 RL 优化摘要的可读性和信息覆盖
     参考: [2301.xxx], [2302.xxx]

  2. **Interactive Summarization Agent**
     用户反馈作为 reward 信号迭代优化摘要

  3. ...
```

### 8. 实验设计助手

```
你: 我的 idea 是 xxx，帮我设计实验方案

Agent: [读取 selected_idea.md 和相关论文]

  ## 实验设计

  ### Datasets
  1. CNN/DailyMail - 标准新闻摘要 (287k samples)
  2. arXiv - 长文档科学论文 (215k samples)
  3. ...

  ### Baselines
  1. BART-large (参考: paper_001.md)
  2. LED (参考: paper_003.md)

  ### Metrics
  - ROUGE-1/2/L
  - BERTScore
  - 人工评估: 流畅度、信息覆盖

  ### Ablation Studies
  1. 去掉 xxx 模块
  2. ...
```

---

## 工作空间结构

```
~/.openclaw/workspace/projects/
├── .active                      # 当前项目 ID
├── nlp-summarization/           # 项目 A
│   ├── project.json             # 元数据
│   ├── task.json                # 任务定义
│   ├── survey/                  # /literature-survey 产出
│   │   ├── search_terms.json
│   │   └── report.md
│   ├── papers/                  # 下载的论文
│   │   ├── _downloads/          # 原始文件
│   │   ├── _meta/               # 元数据 JSON
│   │   └── {direction}/         # 按方向聚类
│   ├── repos/                   # 克隆的仓库
│   ├── notes/                   # /research-survey: 逐篇深度笔记
│   │   └── paper_{arxiv_id}.md
│   ├── survey_res.md            # /research-survey: 方法对比
│   ├── plan_res.md              # /research-plan: 实现计划
│   ├── project/                 # /research-implement: ML 代码
│   │   ├── model/
│   │   ├── data/
│   │   ├── run.py
│   │   └── requirements.txt
│   ├── ml_res.md                # /research-implement: 执行报告
│   ├── iterations/              # /research-review: 审查报告
│   │   └── judge_v*.md
│   ├── experiment_res.md        # /research-experiment: 最终结果
│   └── ideas/                   # 生成的想法
│       ├── idea_1.md
│       ├── idea_2.md
│       └── selected_idea.md     # 最佳想法
└── another-project/
```

---

## 配置

安装后插件自动启用。可以在 `~/.openclaw/openclaw.json` 中自定义设置：

```json
{
  "plugins": {
    "entries": {
      "scientify": {
        "enabled": true,
        "workspaceRoot": "~/my-research",
        "defaultMaxPapers": 15
      }
    }
  }
}
```

### 插件管理

```bash
# 列出已安装插件
openclaw plugins list

# 禁用插件
openclaw plugins disable scientify

# 启用插件
openclaw plugins enable scientify

# 更新到最新版本
openclaw plugins update scientify
```

---

## 已知限制

### Sandbox 与 GPU

`research-pipeline` skill 的代码执行步骤取决于你的 OpenClaw agent 配置：

- 如果 `sandbox.mode: "off"`（CLI 默认），命令直接在主机执行
- 当前 sandbox **不支持** GPU（`--gpus`）和自定义共享内存（`--shm-size`）

对于需要 GPU 加速的 ML 训练：
1. 在 sandbox 外运行（配置 agent `sandbox.mode: "off"`）
2. 使用云 GPU 实例
3. 等待 OpenClaw 添加 GPU 支持

---

## 开发

参见 [CLAUDE.md](./CLAUDE.md) 了解版本更新流程和贡献指南。

---

## License

MIT

## Author

tsingyuai
