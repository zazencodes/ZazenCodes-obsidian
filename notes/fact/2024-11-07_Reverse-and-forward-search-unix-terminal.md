---
date: 2024-11-07
tags:
    - fact
hubs:
    - "[[linux]]"
    = "[[macos]]"
---

# Reverse and forward search unix terminal

To reverse search, type

```bash
<C-r>
```

Then enter your search term, and press `<C-r>` to continue searching through your history

If you need to go back forward (e.g. you passed what you need) then you can go forward with

```bash
<C-s>
```

However this does not always work. You might need to disable terminal flow:

```bash
stty -ixon
```
