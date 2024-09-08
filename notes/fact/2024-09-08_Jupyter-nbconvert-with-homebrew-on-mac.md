---
date: 2024-09-08
tags:
    - fact
hubs:
    - "[[python]]"
---

# Jupyter nbconvert with homebrew on mac


```bash
# Usage e.g. convert to python file
jupyter nbconvert notebooks/dev.ipynb --to script --no-input


# Fix for error: ValueError: No template sub-directory with name 'latex' found in the following paths
sudo ln -s /opt/homebrew/share/jupyter /usr/local/share/jupyter

```
