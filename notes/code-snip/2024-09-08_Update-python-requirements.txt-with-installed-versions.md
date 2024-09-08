---
date: 2024-09-08
tags:
    - code-snip
hubs:
    - "[[python]]"
---

# Update python requirements.txt with installed versions

```bash
#!/bin/bash

# Install as "update_py_requirements_versions" in e.g. ~/bin

# Print before updating, in case recovery is needed
echo "requirements.txt before update:"
cat requirements.txt

# Overwrite with library versions from pip freeze
# Assume using virtual environment named "venv"
packages=$(< requirements.txt)
regex=$(printf '%s|' "${packages[@]}" | tr '\n' '|' | sed 's/|$//' | sed 's/==[^|]*//g')
venv/bin/pip freeze | grep -E "$regex" > requirements.txt

# Print after updating
echo "Updated requirements.txt with package versions that are installed in venv:"
cat requirements.txt
```

