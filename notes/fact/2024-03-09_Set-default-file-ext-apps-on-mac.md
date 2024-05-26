---
date: 2024-03-09
tags:
    - fact
hubs:
    - "[[macos]]"
---

# Set default file ext apps on mac

```bash
# Get app ID
osascript -e 'id of app "TextMate"'

# Set default app, e.g. for JSON files
duti -s com.macromates.TextMate .json all

# Install
brew install duti

```

