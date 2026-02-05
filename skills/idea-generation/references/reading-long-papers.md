# Reading Long Papers Strategy

For papers >50KB or >15k tokens, use chunked reading.

## Step 1: Structure Scan

```bash
ls -la $WORKSPACE/papers/{arxiv_id}/
wc -l $WORKSPACE/papers/{arxiv_id}/*.tex
```

## Step 2: Chunked Reading

Use Read tool with `offset` and `limit`:

```
Tool: Read
Arguments:
  file_path: "$WORKSPACE/papers/2404.04429/main.tex"
  offset: 1
  limit: 500    # First 500 lines
```

## Priority Sections

| Priority | Section | Why |
|----------|---------|-----|
| 1 | Abstract | Core contribution |
| 2 | Method | Technical details |
| 3 | Experiments | Results |
| 4 | Conclusion | Limitations |

## Skip These

- Appendix, Acknowledgments
- Detailed hyperparameter tables

## Quick Extraction

```bash
grep -n "\\\\section{" main.tex
sed -n '/\\begin{abstract}/,/\\end{abstract}/p' main.tex
```
