---
date: 2024-04-09
tags:
    - code-snip
hubs:
    - "[[bash]]"
---

# Concatenate a set of CSVs with the same header in bash


```bash
awk '(NR == 1) || (FNR > 1)' file*.csv > bigfile.csv
```
