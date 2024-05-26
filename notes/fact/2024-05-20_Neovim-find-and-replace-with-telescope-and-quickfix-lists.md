---
date: 2024-05-20
tags:
    - fact
hubs:
    - "[[vim]]"
---

# Neovim find and replace with telescope and quickfix lists

1. Search in telescope
2. Type `C-q`
3. Type `:cdo s/old_pattern/new_pattern/gc` and confirm each with y
4. Save all files `:wa`
5. Close quickfix list with `:cclose`

