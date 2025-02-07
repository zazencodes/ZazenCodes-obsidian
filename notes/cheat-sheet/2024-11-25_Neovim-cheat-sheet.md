---
date: 2024-11-25
tags:
    - cheat-sheet
hubs:
    - "[[linux]]"
---

# Neovim cheat sheet

## Navigation

### Movement

```bash
H # Moves the cursor to the top of the screen
M # Moves the cursor to the middle of the screen
L # Moves the cursor to the bottom of the screen
```

e.g. workflows for scrolling

#1

```bash
L       # Move cursor to bottom (or use H for top)
jjjj    # Move down some lines (or use k for going up)
M       # Move back to middle of code
```

#2

```bash
}   # Move down through code blocks
)   # Move down through code blocks, to first character on line
]]  # Move down through function definitions
zz  # Center screen
```

### Go to

```bash
# Set local/global mark (replace a/A with any character)
ma/mA

# Go to mark
'a/'A

# Open URL in browser
gx

# Open file path
gf
```

## Find and replace

### Yanked string in one file

1. select the text in visual mode
2. yank the text with y
3. `:%s/<C-r>0/` (literally type ctrl+r 0 and the yanked text will be inserted)

### Many files

1. Search in telescope
2. Type `C-q`
3. Type `:cdo s/old_pattern/new_pattern/gc` and confirm each with y
4. Save all files `:wa`
5. Close quickfix list with `:cclose`

## Select text

```bash
# Select inside quotes (shortcuts)
vib/viB

# Select last visual/insert
gv/gi
```

## Format


```bash
# Replace tabs with spaces
:retab

# Python file
:%!black . # UNTESTED

# Format JSON file
:%!jq .

# Format JS file
:%!node_modules/.bin/prettier . -w # UNTESTED

# Fold JSON
zc # close a fold
zo # open a fold
za # toggle a fold
zR # open all folds
zM # close all folds
```



## Plugins

### Nvim tree

```sh
# Show / hide tree
<leader>e

# Show keymaps
g?
```

### Dadbod-ui

```bash
# Launch / close
:DBUIToggle

# Show save location
:echo g:db_ui_save_location

# Configure database connections
vim ~/.local/share/db_ui/

# Execute query
<leader>S

# Query results are persisted in temp storage until vim is quit
```


### Telescope

```sh
# Add all to qfix list
<C-q>

# Add selected to qfix list
# Select with tab
# M = meta key (option for me)
<M-q>
```

## Misc

### Quickfix list

```bash
# Open all files in quickfix list for editing
:cdo edit

# Open/close qf list
:copen
:ccl

# Clear qf list
:cexpr []
```

### Buffers

```bash
# Close all buffers
:%bd
```

### Refresh linter
```bash
:e
```

### Help

```bash
# Print value of vim variable
:echo g:db_ui_save_location
```

### Quit

```bash
ZZ
```


