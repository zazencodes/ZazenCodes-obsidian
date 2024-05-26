---
date: 2024-02-27
tags:
    - fact
hubs:
    - "[[python]]"
    - "[[vim]]"
---

# vim python

## Using virtual environments

Activate it before opening vim

```bash
source venv/bin/activate
vim

```

## Show vim python version

```
:!which python
:python import sys; print(sys.version)
```
Check vim python support for various libraries
```
:checkhealth
```

### black

Running `:!black %` formats the current buffer by calling `black`. On my MacOS this uses the homebrew version of `black`

