---
date: 2024-03-17
tags:
    - fact
hubs:
    - "[[python]]"
urls:
    - https://stackoverflow.com/questions/15268953/how-to-install-python-package-from-github
---

# Python install library from github repository

e.g.
```
pip install git+https://github.com/jkbr/httpie.git#egg=httpie
```

Include `egg=<projectname>` to explicitly name the project so that pip can track metadata for it without having to have run the setup.py script.


