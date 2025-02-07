---
date: 2025-02-04
tags:
    - fact
hubs:
    - "[[python]]"
urls:
    - https://github.com/pypa/pipx
---

# pipx for installing and running applications with pip

## What is it?

pipx is a tool to install and run Python applications in isolated environments. It's like `brew` or `npm install -g` but for Python packages, making it safer and cleaner to install Python CLI applications globally.


* Installs each application in its own isolated virtual environment
* Allows global access to applications without polluting system Python
* Makes it easy to upgrade, uninstall, and manage Python applications
* Perfect for CLI tools like `black`, `poetry`, `youtube-dl`, etc.
* Ensures dependencies don't conflict between applications


## Usage Examples

```bash
pipx install llm
```
