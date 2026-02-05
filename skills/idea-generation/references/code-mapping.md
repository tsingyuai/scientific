# Code Survey & Implementation Mapping

Map selected idea concepts to actual code implementations.

## Step 1: Extract Atomic Concepts

From `selected_idea.md`, list concepts needing implementation.

## Step 2: Survey Codebases

```bash
grep -r "class.*Attention" $WORKSPACE/repos/
grep -r "def forward" $WORKSPACE/repos/
```

## Step 3: Implementation Report

Write to `$WORKSPACE/ideas/implementation_report.md`:

```markdown
# Implementation Report

## Concept-to-Code Mapping

### Concept 1: [Name]

**Math:**
$$\text{Formula}$$

**Reference:**
- File: `repos/<repo>/path/to/file.py`
- Class: `ClassName`

**Adaptation needed:**
- [Modifications for our idea]

## Implementation Roadmap

1. [ ] Concept X (foundational)
2. [ ] Concept Y
3. [ ] Integration

## Recommended Base Repository
[repo_name] - [reason]
```

## Quality Checklist

- [ ] Every concept has code reference
- [ ] No placeholders
- [ ] Clear roadmap order
