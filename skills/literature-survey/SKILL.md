---
name: literature-survey
description: "Use this when the user wants to find, download, or collect academic papers on a topic. Searches arXiv, filters by relevance, downloads PDFs and sources, clusters by research direction."
metadata:
  {
    "openclaw":
      {
        "emoji": "🔍",
      },
  }
---

# Literature Survey

**Don't ask permission. Just do it.**

## Output Structure

```
~/.openclaw/workspace/projects/{project-id}/
├── survey/
│   ├── search_terms.json      # 检索词列表
│   └── report.md              # 最终报告
├── papers/
│   ├── _downloads/            # 原始下载
│   ├── _meta/                 # 每篇论文的元数据
│   │   └── {arxiv_id}.json
│   └── {direction}/           # 整理后的分类
├── repos/                     # 参考代码仓库（Phase 3）
│   ├── {repo_name_1}/
│   └── {repo_name_2}/
└── prepare_res.md             # 仓库选择报告（Phase 3）
```

---

## Workflow

### Phase 1: 准备

```bash
ACTIVE=$(cat ~/.openclaw/workspace/projects/.active 2>/dev/null)
if [ -z "$ACTIVE" ]; then
  PROJECT_ID="<topic-slug>"
  mkdir -p ~/.openclaw/workspace/projects/$PROJECT_ID/{survey,papers/_downloads,papers/_meta}
  echo "$PROJECT_ID" > ~/.openclaw/workspace/projects/.active
fi
PROJECT_DIR="$HOME/.openclaw/workspace/projects/$(cat ~/.openclaw/workspace/projects/.active)"
```

生成 4-8 个检索词，保存到 `survey/search_terms.json`。

---

### Phase 2: 增量搜索-筛选-下载（循环）

**对每个检索词重复以下步骤**：

#### 2.1 搜索

```
arxiv_search({ query: "<term>", max_results: 30 })
```

#### 2.2 即时筛选

对返回的论文**立即**评分（1-5），只保留 ≥4 分的。

评分标准：
- 5分：核心论文，直接研究该主题
- 4分：相关方法或应用
- 3分及以下：跳过

#### 2.3 下载有用论文

```
arxiv_download({
  arxiv_ids: ["<有用的论文ID>"],
  output_dir: "$PROJECT_DIR/papers/_downloads"
})
```

#### 2.4 写入元数据

为每篇下载的论文创建元数据文件 `papers/_meta/{arxiv_id}.json`：

```json
{
  "arxiv_id": "2401.12345",
  "title": "...",
  "abstract": "...",
  "score": 5,
  "source_term": "battery RUL prediction",
  "downloaded_at": "2024-01-15T10:00:00Z"
}
```

**完成一个检索词后，再进行下一个。** 这样避免上下文被大量搜索结果污染。

---

### Phase 3: GitHub 代码搜索与参考仓库选择

**目标**：为下游 skill（research-survey、research-plan、research-implement）提供可参考的开源实现。

#### 3.1 选择高分论文

读取 `papers/_meta/` 下得分 ≥4 的论文，选出 **Top 5** 最相关论文。

#### 3.2 搜索参考仓库

对每篇选中论文，用以下关键词组合搜索 GitHub 仓库：
- 论文标题 + "code" / "implementation"
- 核心方法名 + 作者名
- 论文中提到的数据集名 + 任务名

使用 `web_search` 或 `exec` 调用 GitHub API：
```bash
# 示例：
web_search({ query: "{paper_title} implementation github" })
```

#### 3.3 筛选与 clone

对搜索到的仓库，评估：
- Star 数（建议 >100）
- 代码质量（有 README、有 requirements.txt、代码结构清晰）
- 与论文的匹配度（README 中引用了论文 / 实现了论文中的方法）

选择 **3-5 个**最相关的仓库，clone 到 `repos/`：

```bash
mkdir -p "$PROJECT_DIR/repos"
cd "$PROJECT_DIR/repos"
git clone --depth 1 <repo_url>
```

#### 3.4 写入选择报告

创建 `$PROJECT_DIR/prepare_res.md`：

```markdown
# 参考仓库选择

| 仓库 | 对应论文 | Stars | 选择理由 |
|------|----------|-------|----------|
| repos/{repo_name} | {paper_title} (arxiv:{id}) | {N} | {理由} |

## 各仓库关键文件

### {repo_name}
- **模型实现**: `model/` 或 `models/`
- **训练脚本**: `train.py` 或 `main.py`
- **数据加载**: `data/` 或 `dataset.py`
- **核心文件**: `{关键文件路径}` — {描述}
```

**如果搜不到相关仓库**，在 `prepare_res.md` 中注明"无可用参考仓库"，后续 skill 将不依赖代码映射。

---

### Phase 4: 分类整理

所有检索词和代码搜索完毕后：

#### 4.1 读取所有元数据

```bash
ls $PROJECT_DIR/papers/_meta/
```

读取所有 `.json` 文件，汇总论文列表。

#### 4.2 聚类分析

根据论文的标题、摘要、来源检索词，识别 3-6 个研究方向。

#### 4.3 创建文件夹并移动

```bash
mkdir -p "$PROJECT_DIR/papers/data-driven"
mv "$PROJECT_DIR/papers/_downloads/2401.12345" "$PROJECT_DIR/papers/data-driven/"
```

---

### Phase 5: 生成报告

创建 `survey/report.md`：
- 调研概要（检索词数、论文数、方向数）
- 各研究方向概述
- Top 10 论文
- **参考仓库摘要**（引用 prepare_res.md）
- 建议阅读顺序

---

## 关键设计

| 原则 | 说明 |
|------|------|
| **增量处理** | 每个检索词独立完成搜索→筛选→下载→写元数据，避免上下文膨胀 |
| **元数据驱动** | 分类基于 `_meta/*.json`，不依赖内存中的大列表 |
| **文件夹即分类** | 聚类结果通过 `papers/{direction}/` 体现，无需额外 JSON |

## Tools

| Tool | Purpose |
|------|---------|
| `arxiv_search` | 搜索论文（无副作用） |
| `arxiv_download` | 下载 .tex/.pdf（需绝对路径） |
