---
date: 2024-05-30
tags:
    - fact
hubs:
    - "[[linux]]"
    - "[[macos]]"
---

# Check Overall Disk Usage

Run this to see where space is being used:

```bash
sudo du -h --max-depth=1 / | sort -hr
```


# File size

Total size of matched files
```bash
du -sch MATCH_PATTERN
```

Total disk storage
```bash
df -h
```


## List file size recursively


```bash
# List all disk usage
df -h

# Size of all files in current dir
du -sh

# Recursive size of all folders in dir
du -sh *
```

