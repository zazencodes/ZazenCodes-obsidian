---
date: 2024-03-11
tags:
    - cheat-sheet
hubs:
    - "[[linux]]"
    - "[[macos]]"
urls:
    - https://www.linode.com/docs/guides/using-jq-to-process-json-on-the-command-line/
---

# jq command line JSON tool

Pretty print json
```bash
jq . file.json
```

Top-level keys
```bash
jq keys file.json
```

Get keys
```bash
jq .key_name file.json
```

Get items
```bash
jq '.key_name.[0]' file.json
```

Iter over items
```bash
jq '.key_name.[].subkey_name' file.json
```

Iter over items
```bash
jq '.key_name.[3:5].subkey_name' file.json
```

Operators
```bash
jq '.key_name.[].subkey_name | [length / max / keys]' file.json
jq 'map(.key_name) | unique' file.json
```

Fitlers
```bash
jq '.[] | select (.key_name > 18)' file.json
```



