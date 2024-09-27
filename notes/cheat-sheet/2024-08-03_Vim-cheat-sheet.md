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

## Quit

```bash
ZZ
```
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
# With telescope open, populate quickfix list with results
<C-q>
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
