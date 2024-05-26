---
date: 2021-10-13
tags:
    - fact
hubs:
    - "[[algorithms]]"
    - "[[leetcode]]"
urls:
    - https://leetcode.com/problems/longest-common-subsequence/
---

# Longest shared subsequence

Learnings

- Draw out problem in editor. This helps since you're a visual learner. Rely on examples

e.g. 

```
"""


  a b c d e e
a 1 1 1 1 1 1

c 1 1 2 2 2 2

e 1 1 2 2 3 3

e 1 1 2 2 3 4 <- answer


"""
```

- Solution:

```
class Solution:
    def longestCommonSubsequence(self, text1, text2):
        if len(text1) == 0 or len(text2) == 0:
            return 0

        res = [[0]*(len(text1)+1) for _ in range(len(text2)+1)]

        print(res)

        for j, tj in enumerate(text1):
            for i, ti in enumerate(text2):
                if ti == tj:
                    res[i+1][j+1] = res[i][j] + 1
                else:
                    res[i+1][j+1] = max(res[i+1][j], res[i][j+1])

        print(res)

        return res[-1][-1]


Solution().longestCommonSubsequence("abcddee", "acdee")
```

