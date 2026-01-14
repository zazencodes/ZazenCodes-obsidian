---
date: 2026-01-14
tags:
    - fact
hubs:
    - "[[math-stats]]"
---

# Collision Probability Analysis for ID generation

## Overview

We are considering the collision probability for a function that generates a 12-character identifier by hashing an input string using SHA-256, encoding it via URL-safe Base64, and stripping non-alphanumeric characters to return the first 12 characters.

```python
import hashlib
import base64
import re

def generate_input_string_id(input_string: str) -> str:
    """
    Hash a prompt or persona string into a 12-character ID.

    Args:
        input_string: Persona or prompt string

    Returns:
        12-character alphanumeric ID
    """
    length = 12
    digest = hashlib.sha256(input_string.encode("utf-8")).digest()
    id_with_special_chars = base64.urlsafe_b64encode(digest).decode("utf-8")
    cleaned_id = re.sub(r"[^A-Za-z0-9]", "", id_with_special_chars)
    if len(cleaned_id) < length:
        raise ValueError(f"Cannot assign {length} character ID to prompt")
    return cleaned_id[:12]
```


### ID Specifications
- **Alphabet Size ($k$):** 62 (A-Z, a-z, 0-9)
- **ID Length ($L$):** 12
- **Total Search Space ($M$):** $62^{12} \approx 3.22 \times 10^{21}$ combinations

## Mathematical Principles

The collision risk is analyzed using two primary mathematical frameworks: **Classical Probability** for a single pair and the **Birthday Problem** for a population.

### 1. The Collision of Two Specific Strings
If you have one existing ID, the probability that a second generated ID matches it exactly is 1 divided by the total number of possible IDs.
$$P = \frac{1}{M} = \frac{1}{62^{12}} \approx 3.1 \times 10^{-22}$$

### 2. The Birthday Paradox (Population Collision)
When generating $n$ strings, we check if **any** ID in the set matches **any other** ID. As $n$ increases, the number of possible pairs increases quadratically.

The probability of at least one collision $P(n)$ is derived from the **Taylor expansion approximation** of the Birthday Problem:
$$P(n) \approx 1 - e^{-\frac{n^2}{2M}} \approx \frac{n^2}{2M}$$

**Why $n^2$?** This represents the growth of possible pairings within the dataset. While the ID space is vast, the number of opportunities for a collision grows much faster than the number of IDs themselves.

## Statistical Risk Table

| Total IDs ($n$) | Collision Probability ($P$) | Practical Context |
| :--- | :--- | :--- |
| **2** | $3.1 \times 10^{-22}$ | Statistically impossible |
| **1,000** | $1.5 \times 10^{-16}$ | Negligible |
| **1,000,000 (1M)** | $1.5 \times 10^{-10}$ | ~1 in 6.4 billion (Powerball odds) |
| **100,000,000 (100M)** | $1.5 \times 10^{-6}$ | ~1 in 640,000 |
| **1,000,000,000 (1B)** | $1.5 \times 10^{-4}$ | ~1 in 6,400 |

## Summary Findings
* **Safety Threshold:** This approach is highly resilient for datasets up to **100 million records**. At this scale, the risk of a collision is significantly lower than the probability of hardware-level data corruption.
* **Scaling Warning:** Because the risk grows with $n^2$, the probability of collision becomes statistically significant (~1 in 6,400) once the database reaches **1 billion records**.
* **Recommendation:** For systems targeting multi-billion record datasets, it is recommended to increase the slice length to **16 characters** to expand the search space to $62^{16}$.


