---
date: 2024-08-03
tags:
    - cheat-sheet
hubs:
    - "[[vim]]"
---

# Vim cheat sheet


## Buffers

```bash
# Close all buffers
:%bd
```

## Select text

```bash
# Select inside quotes (shortcuts)
vib/viB
```

## Go to

```bash
# Open URL in browser
gx

# Open file path
gf

# Set local/global mark (replace a/A with any character)
ma/mA

# Go to mark
'a/'A
```

## Find and replace

Replace string in one file:
- select the text in visual mode
- yank the text with y
- `:%s/<C-r>0/` (literally type ctrl+r 0 and the yanked text will be inserted)

## Quickfix list

```bash
# Open/close qf list
:copen
:ccl

# Clear qf list
:cexpr []
```

Open multiple files with telescope search

```bash
# With telescope open, populate quickfix list with all results
<C-q>
# With telescope open, populate quickfix list with selected results (use Tab to select)
# M = meta key (option key on Mac, if configured in terminal)
<M-q>
# Open all files in quickfix list for editing
:cdo edit
```

## Format

```bash
# Format JSON file
:%!jq .
```

### Refresh linter
```bash
:e
```

## Help

```bash
# Print value of vim variable
:echo g:db_ui_save_location
```

## Quit

```bash
ZZ
```
