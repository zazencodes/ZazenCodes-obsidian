---
date: 2024-03-05
tags:
    - fact
hubs:
    - "[[git]]"
urls:
    - https://stackoverflow.com/questions/18216991/create-a-tag-in-a-github-repository
---

# Git project tags

## Creating tags

To create a tag on your current branch:
```
git tag <tagname> [-a "Tag description"]
```

e.g.

```
git tag v0.1
```
You will need to explicitly say that you want to push your tags to your remote repo:

```
git push origin --tags
```

Or if you just want to push a single tag:

```
git push origin <tag>
```


