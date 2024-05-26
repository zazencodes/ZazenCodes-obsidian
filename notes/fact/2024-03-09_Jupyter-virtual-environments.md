---
date: 2020-03-09
tags:
    - fact
hubs:
    - "[[python]]"
urls:
    - https://janakiev.com/blog/jupyter-virtual-envs/
---

# Jupyter virtual environments

In order to make regular python virtual environments available in Jupyter Notebooks you need to install the `ipykernel` lib as follows:

```bash
python -m venv myvenv
myvenv/bin/pip install --user ipykernel
myvenv/bin/python -m ipykernel install --user --name=myenv

# To remove
jupyter kernelspec list
jupyter kernelspec uninstall myenv
```
