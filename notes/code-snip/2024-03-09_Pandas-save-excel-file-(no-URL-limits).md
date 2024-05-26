---
date: 2022-03-09
tags:
    - code-snip
hubs:
    - "[[python]]"
    - "[[pandas]]"
---

# Pandas save excel file (no URL limits)

```python
with pd.ExcelWriter('file.xlsx', options={'strings_to_urls': False}) as f:
    df.to_excel(writer, 'content', index=False)
    df_links.to_excel(writer, 'links', index=False)
    writer.save()
```

