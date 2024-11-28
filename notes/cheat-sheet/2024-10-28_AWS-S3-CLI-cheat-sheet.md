---
date: 2024-10-28
tags:
    - cheat-sheet
hubs:
    - "[[devops]]"
    - "[[data-engineering]]"
---

# AWS S3 CLI cheat sheet

## Tips

- Add --dryrun flag to any command

## Copy

### Bucket folder recursively

```bash
aws s3 cp --recursive s3://[BUCKET_NAME]/[PATH] .
```

## List


```bash
# By lastmod
aws s3api list-objects-v2 --bucket [BUCKET_NAME] --prefix [BUCKET_FOLDER]/ --query 'Contents[].[Key, LastModified]' --output json | jq 'sort_by(.[1]) | reverse | .[].0'

# Total size of files with prefix (in GiB)
aws s3api list-objects --bucket [BUCKET_NAME] --prefix [BUCKET_FOLDER]/ --query "Contents[].Size" --output json | jq 'add / (1024 * 1024 * 1024)'
```


