---
date: 2024-09-10
tags:
    - cheat-sheet
hubs:
    - "[[data-engineering]]"
    - "[[vim]]"
---

# Neovim database tools


## Dadbod-ui

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

