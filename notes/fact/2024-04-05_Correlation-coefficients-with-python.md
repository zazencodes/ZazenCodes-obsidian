---
date: 2024-04-05
tags:
    - fact
hubs:
    - "[[python]]"
    - "[[pandas]]"
---

# Correlation coefficients with python

[Pearson correlation coefficients](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient):

```python
df.corr()
```

Visualize with seaborn:

```python

import seaborn as sns
sns.heatmap(df.corr(), annot=True, fmt=".3f")
```

Use `vmin` and `vmax` to set min/max
