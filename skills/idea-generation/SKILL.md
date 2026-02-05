---
name: idea-generation
description: "Generate innovative research ideas from a topic. SEARCHES arXiv/GitHub automatically, downloads papers, analyzes literature, outputs 5 novel ideas with citations. Use for: 找研究方向, 生成创新点, find research gaps, propose new methods. NOT for: summarizing papers (use /write-review-paper), literature survey (use /literature-survey)."
metadata:
  {
    "openclaw":
      {
        "emoji": "💡",
        "requires": { "bins": ["git"] },
      },
  }
---

# Idea Generation

End-to-end workflow for generating innovative research ideas from a research topic. This skill implements a full research idea generation pipeline:

1. Search papers and code repositories
2. Select and download references
3. Analyze literature and codebases
4. Generate multiple ideas
5. Select and enhance the best idea
6. Map to code implementations

---

## ⚠️ CRITICAL: EXECUTION MODE

**AUTONOMOUS EXECUTION**: Execute ALL steps without asking for user confirmation at each step.
- Do NOT ask "要我继续吗?" or "Should I proceed?"
- You MAY spawn subagents for parallel tasks (e.g., downloading multiple papers)
- Only ask user when there's a genuine ambiguity (e.g., which focus area to choose)
- Checkpoints are for YOUR internal verification, not for asking user

**Run the entire workflow from Step 1 to Step 8 automatically.**

---

## ⚠️ CRITICAL: MANDATORY TOOL USAGE

**DO NOT generate ideas from your own knowledge.** All ideas MUST be grounded in actual literature research.

### Blocking Requirements

1. **MUST call `arxiv` tool** to search papers - NO EXCEPTIONS
2. **MUST call `github_search` tool** to find repositories - NO EXCEPTIONS
3. **MUST write `search_results.md`** BEFORE proceeding to idea generation
4. **MUST reference specific papers** (with arXiv IDs) in generated ideas
5. **MUST clone actual repos** before code survey

### Anti-Pattern: DO NOT DO THIS

❌ User asks about "time series forecasting" → Agent immediately lists methods from memory
❌ Agent generates ideas without calling any search tools
❌ Agent skips to idea generation without `search_results.md` existing

### Correct Pattern: DO THIS

✅ User asks about "time series forecasting" → Agent calls `arxiv` tool with query
✅ Agent calls `github_search` tool to find implementations
✅ Agent writes search results to file
✅ Agent reads downloaded papers before generating ideas
✅ Ideas reference specific papers by arXiv ID

---

## Workspace Convention (Project-based)

**IMPORTANT**: Each research topic uses its own project directory. Agent auto-selects or creates projects.

```
~/.openclaw/workspace/
└── projects/
    ├── .active                   # Current project ID (plain text file)
    ├── nlp-summarization/        # Project A
    │   ├── project.json          # Project metadata
    │   ├── task.json             # Research task definition
    │   ├── search_results.md     # Search results
    │   ├── prepare_res.md        # Selected repos summary
    │   ├── papers/               # Downloaded papers
    │   ├── repos/                # Cloned repositories
    │   └── ideas/                # Generated ideas
    ├── image-segmentation/       # Project B
    │   └── ...
    └── ...
```

**All paths are project-relative**: `~/.openclaw/workspace/projects/{project_id}/`

**File existence = step completion.** Skip steps whose output already exists.

---

## Step 0: Auto Project Management (REQUIRED)

**Autonomous - DO NOT ask user for confirmation.**

1. Extract topic from user query → convert to kebab-case ID
2. Check `~/.openclaw/workspace/projects/` for existing match
3. Use existing or create new: `mkdir -p $PROJECT_ID/{papers,repos,ideas}`
4. Update `.active` file and set `$WORKSPACE` path

> 📁 Using project: `{project_id}` (new/existing)

---

## Step 1: Parse Task

Check `$WORKSPACE/task.json`. If missing, extract from user query:

- **domain**: Research domain (e.g., "graph neural networks", "recommendation")
- **focus** (optional): Specific problem or technique
- **date_limit** (optional): Only consider papers before this date

```bash
cat $WORKSPACE/task.json 2>/dev/null || echo "No task.json"
```

Create task.json:
```json
{
  "domain": "graph neural networks",
  "focus": "scalable transformers for node classification",
  "date_limit": "2024-01-01",
  "created": "2024-XX-XX"
}
```

**Output:** `$WORKSPACE/task.json`

---

## Step 2: Search Papers and Code (MANDATORY)

**⚠️ BLOCKING: You MUST complete this step before ANY idea generation.**

### 2.1 ArXiv Search (REQUIRED)

**You MUST call the `arxiv` tool.** Example:

```
Tool: arxiv
Arguments:
  query: "text summarization transformer model"
  max_results: 10
  sort_by: "relevance"
```

If `arxiv` tool is not available, use `WebSearch` with `site:arxiv.org`:
```
Tool: WebSearch
Arguments:
  query: "site:arxiv.org text summarization transformer model"
```

### 2.2 GitHub Search (REQUIRED)

**You MUST call the `github_search` tool or search GitHub.** Example:

```
Tool: github_search
Arguments:
  query: "text summarization pytorch huggingface"
  sort: "stars"
  max_results: 20
```

If `github_search` tool is not available, use `WebSearch`:
```
Tool: WebSearch
Arguments:
  query: "site:github.com text summarization pytorch stars:>100"
```

### 2.3 CHECKPOINT: Verify Search Completed

Before proceeding, confirm:
- [ ] Called arxiv/WebSearch for papers
- [ ] Called github_search/WebSearch for repositories
- [ ] Have at least 5 paper results
- [ ] Have at least 5 repository results

**If search returns 0 results, try different queries. DO NOT proceed without results.**

### 2.4 Compile Results

Write to `$WORKSPACE/search_results.md`:

```markdown
# Search Results

## Task
- Domain: {domain}
- Focus: {focus}
- Date: {date}

## ArXiv Papers Found

| # | Title | ArXiv ID | Year | Relevance |
|---|-------|----------|------|-----------|
| 1 | [Title](pdf_url) | 2401.xxxxx | 2024 | [Why relevant] |
| 2 | ... | ... | ... | ... |

## GitHub Repositories Found

| # | Repository | Stars | Language | Relevance |
|---|------------|-------|----------|-----------|
| 1 | [owner/repo](url) | 1.2k | Python | [Why relevant] |
| 2 | ... | ... | ... | ... |
```

**Output:** `$WORKSPACE/search_results.md`

---

## Step 3: Prepare - Select Repositories

Read search results and select **at least 5** most valuable repositories.

Selection criteria:
- Direct implementation of relevant papers
- High code quality (stars, documentation)
- Active maintenance
- Covers key techniques in the domain

### 3.1 Clone Selected Repos

```bash
mkdir -p $WORKSPACE/repos
cd $WORKSPACE/repos

# For each selected repo:
git clone --depth 1 https://github.com/owner/repo1.git
git clone --depth 1 https://github.com/owner/repo2.git
# ... at least 5 repos
```

### 3.2 Document Selection

Write to `$WORKSPACE/prepare_res.md`:

```markdown
# Selected Reference Codebases

## Selection Rationale
[Why these repos were chosen]

## Repositories

### 1. repo1
- **URL**: https://github.com/owner/repo1
- **Paper**: [Associated paper if any]
- **Key Components**:
  - `model/` - Model architecture
  - `train.py` - Training loop
- **Usage**: [How this will help implement our idea]

### 2. repo2
...

## Reference Papers
Based on these repos, the key papers to read are:
1. [Paper Title 1] - ArXiv: 2401.xxxxx
2. [Paper Title 2] - ArXiv: 2401.xxxxx
...
```

**Output:** `$WORKSPACE/prepare_res.md` + `$WORKSPACE/repos/`

---

## Step 4: Download Papers

For each paper referenced in prepare_res.md, download the source.

**IMPORTANT: Download .tex source, NOT PDF.** .tex files are much easier for AI to read and extract information from.

### 4.1 Download .tex Source (RECOMMENDED - Use arxiv tool)

Use the `arxiv` tool with `download: true` to automatically download and extract .tex sources:

```
Tool: arxiv
Arguments:
  query: "abstractive summarization long document"
  max_results: 10
  download: true
  output_dir: "$WORKSPACE/papers"
```

The tool will:
1. Search for papers matching your query
2. Download .tex source from `https://arxiv.org/src/{arxiv_id}`
3. Extract tar.gz archives automatically
4. Fall back to PDF if .tex is unavailable
5. Return a `downloads` array showing what was downloaded

**Output format:**
```json
{
  "papers": [...],
  "downloads": [
    {"arxiv_id": "2404.04429", "format": "tex", "files": ["main.tex", "methods.tex"]},
    {"arxiv_id": "2308.03664", "format": "pdf", "files": ["2308.03664.pdf"], "error": "tex unavailable"}
  ],
  "output_dir": "$WORKSPACE/papers"
}
```

### 4.2 Manual Download (Fallback)

If the arxiv tool is unavailable, use bash:
```bash
mkdir -p $WORKSPACE/papers/{arxiv_id}
cd $WORKSPACE/papers/{arxiv_id}
curl -L "https://arxiv.org/src/{arxiv_id}" -o source.tar.gz
tar -xzf source.tar.gz 2>/dev/null || mv source.tar.gz main.tex
```

### 4.3 Document Downloads

Write to `$WORKSPACE/papers/download_log.md`:

```markdown
# Downloaded Papers

| ArXiv ID | Title | Format | Status |
|----------|-------|--------|--------|
| 2404.04429 | Physics-Informed ML for Battery... | .tex | ✓ |
| 2308.03664 | Two-stage Early Prediction... | .tex | ✓ |
| 2401.99999 | Some Other Paper | .pdf | ✓ (tex unavailable) |
```

**Output:** `$WORKSPACE/papers/`

---

## Step 5: Generate Ideas (5 Ideas)

**⚠️ BLOCKING: DO NOT start this step unless Steps 2-4 are complete.**

### Pre-requisite Checkpoint

Before generating ANY ideas, verify these files exist:
- [ ] `$WORKSPACE/search_results.md` - search results from Step 2
- [ ] `$WORKSPACE/prepare_res.md` - selected repos from Step 3
- [ ] At least 3 papers downloaded in `$WORKSPACE/papers/`

**If any file is missing, GO BACK and complete the previous steps.**

This is the core intellectual step. Generate **exactly 5 distinct innovative ideas**.

**IMPORTANT: Ideas must be grounded in the literature you just read. Each idea MUST:**
- Reference at least 2 specific papers by arXiv ID
- Identify specific limitations from those papers
- Propose improvements based on gaps found in the literature

### 5.1 Analyze Literature First (REQUIRED)

For each paper in `papers/`:
1. Read thoroughly (especially: abstract, method, experiments, limitations)
2. Extract: core contribution, math formulas, limitations, future work
3. Note connections to other papers

**Long Papers (>50KB):** See `references/reading-long-papers.md` for chunked reading strategy.

For each repo in `repos/`:
1. Understand structure: `gen_code_tree_structure` equivalent
2. Identify key implementations
3. Note reusable components

### 5.2 Identify Research Gaps

Look for:
- Common limitations across papers
- Unexplored combinations of techniques
- Scalability issues
- Assumptions that could be relaxed

### 5.3 Generate 5 Ideas

Create `$WORKSPACE/ideas/idea_1.md` through `idea_5.md` using template in `references/idea-template.md`.

**Key requirements:**
- Each idea must cite ≥2 papers by arXiv ID
- Use different strategies (see template): combination, simplification, generalization, constraint relaxation, architecture innovation

**❌ REJECTED if:** No arXiv IDs cited, or ideas not grounded in literature

**Output:** `$WORKSPACE/ideas/idea_1.md` through `idea_5.md`

---

## Step 6: Select and Enhance Best Idea

### 6.1 Evaluate All Ideas

Score each idea on Novelty/Feasibility/Impact (1-5). Select highest total.

### 6.2 Enhance Selected Idea

Create `$WORKSPACE/ideas/selected_idea.md` with:
- Detailed math (loss functions, gradients)
- Architecture choices
- Hyperparameters
- Implementation roadmap
- Failure modes & mitigations

**Output:** `$WORKSPACE/ideas/selected_idea.md`

---

## Step 7: Code Survey - Map Idea to Implementations

Map each **atomic concept** in the selected idea to code in reference repos.

See `references/code-mapping.md` for detailed template.

**Quick steps:**
1. Extract atomic concepts from `selected_idea.md`
2. Search repos: `grep -r "class.*Attention" $WORKSPACE/repos/`
3. Document mapping to `$WORKSPACE/ideas/implementation_report.md`

**Output:** `$WORKSPACE/ideas/implementation_report.md`

---

## Step 8: Final Summary

Create `$WORKSPACE/ideas/summary.md` with:
- Task overview (domain, focus)
- Resources gathered (papers, repos count)
- All 5 ideas with scores
- Selected idea details
- Next steps: `/research-pipeline` or manual implementation

**Output:** `$WORKSPACE/ideas/summary.md`

---

## Quality Checklist

Before completing, verify:

- [ ] At least 5 repos cloned in `repos/`
- [ ] At least 3 papers downloaded in `papers/`
- [ ] All 5 ideas are substantially different
- [ ] Selected idea has complete math formulations
- [ ] Implementation report covers ALL atomic concepts
- [ ] Each concept has actual code reference (not placeholder)
- [ ] Evaluation plan has specific datasets and metrics

---

## Integration with Other Skills

**After idea-generation:**
- `/research-pipeline` → Implement the selected idea

**To gather more resources:**
- `/literature-survey` → Comprehensive paper collection
- `/write-review-paper` → Synthesize into review

---

## Commands

| User Says | Action |
|-----------|--------|
| "Generate research ideas for NLP" | Full workflow (Steps 1-8) |
| "Search papers on X" | Steps 1-2 only |
| "I have papers, generate ideas" | Skip to Step 5 |
| "Enhance this idea: ..." | Skip to Step 6-7 |
| "Map this idea to code" | Step 7 only |

---

## Batch Processing Rule

If more than 10 papers/repos to analyze:
1. First pass: Quick scan all (abstract/README only)
2. Select top 5-7 for deep analysis
3. Generate ideas from deep analysis

Do NOT process all resources with full detail - context will overflow.
