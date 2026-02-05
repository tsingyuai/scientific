# Workspace Directory Specification

All Scientify skills share a unified project-based workspace structure.

## Base Path

```
~/.openclaw/workspace/projects/
├── .active                      # Current project ID (plain text)
├── {project-id}/                # Each research topic has its own project
│   └── ...
└── {another-project}/
```

## Project Structure

```
~/.openclaw/workspace/projects/{project-id}/
├── project.json                 # Project metadata
├── task.json                    # Research task definition
│
├── survey/                      # /literature-survey outputs
│   ├── search_terms.json        # Generated search keywords
│   ├── raw_results.json         # All search results
│   ├── filtered_papers.json     # Papers with relevance scores
│   ├── clusters.json            # Clustered by research direction
│   └── report.md                # Final survey report
│
├── papers/                      # Downloaded paper sources
│   ├── {direction-1}/           # Organized by cluster
│   │   ├── paper_list.md
│   │   └── {arxiv_id}/          # .tex source files
│   ├── {direction-2}/
│   └── uncategorized/
│
├── repos/                       # Cloned reference repositories
│   ├── {repo-name-1}/
│   └── {repo-name-2}/
│
├── ideas/                       # /idea-generation outputs
│   ├── gaps.md                  # Identified research gaps
│   ├── idea_1.md ... idea_5.md  # Generated ideas
│   ├── selected_idea.md         # Enhanced best idea
│   ├── implementation_report.md # Code mapping
│   └── summary.md               # Final summary
│
├── review/                      # /write-review-paper outputs
│   ├── reading_plan.md          # Prioritized reading list
│   ├── notes/                   # Per-paper reading notes
│   │   └── {paper_id}.md
│   ├── comparison.md            # Method comparison table
│   ├── timeline.md              # Research timeline
│   ├── taxonomy.md              # Classification system
│   ├── draft.md                 # Survey paper draft
│   └── bibliography.bib         # References
│
├── plan_res.md                  # /research-pipeline: implementation plan
├── project/                     # /research-pipeline: code implementation
│   ├── model/
│   ├── data/
│   ├── training/
│   ├── testing/
│   ├── run.py
│   └── requirements.txt
├── iterations/                  # Review iterations
│   ├── judge_v1.md
│   └── judge_v2.md
└── experiment_res.md            # Final results
```

## Conventions

### File Existence = Step Completion

Check output file before executing any step. If exists, skip.

Enables:
- **Crash recovery**: resume from last completed step
- **Incremental progress**: rerunning skips completed work
- **Transparency**: inspect progress by listing directory

### Project Metadata

**project.json:**
```json
{
  "id": "battery-rul-prediction",
  "name": "Battery RUL Prediction",
  "created": "2024-01-15T10:00:00Z",
  "topics": ["battery", "remaining useful life", "prediction"]
}
```

**task.json:**
```json
{
  "domain": "battery health",
  "focus": "RUL prediction using transformer",
  "date_limit": "2024-01-01",
  "created": "2024-01-15"
}
```

### Immutability

Once written, do NOT modify outputs unless user explicitly asks.
Exception: `project/` is mutable during implement-review-iterate loop.

### Active Project

```bash
# Read active project
cat ~/.openclaw/workspace/projects/.active

# Set active project
echo "battery-rul-prediction" > ~/.openclaw/workspace/projects/.active

# Set $WORKSPACE variable
WORKSPACE=~/.openclaw/workspace/projects/$(cat ~/.openclaw/workspace/projects/.active)
```

## Skill Outputs Summary

| Skill | Primary Outputs |
|-------|-----------------|
| `/literature-survey` | `survey/`, `papers/` |
| `/idea-generation` | `ideas/` |
| `/write-review-paper` | `review/` |
| `/research-pipeline` | `project/`, `iterations/`, `experiment_res.md` |
