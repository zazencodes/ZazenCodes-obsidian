---
date: 2025-02-04
tags:
    - fact
hubs:
    - "[[bash]]"
---

# llm cat dir alias command

Added to `~/.zshrc`

```bash
alias llm_cat_dir="find . -maxdepth 1 -type f | xargs -I {} sh -c 'echo \"\n=== {} ===\n\"; cat {}'"
```

E.g. run like this:

```bash
cd project-dir
llm_cat_dir | llm 'Summarize this codebase'
```

This could be a good pattern to continue using:

**alias files with underscores**

And I can group them by use case, e.g. `llm_*`, `bq_`, `aws_`, etc...

