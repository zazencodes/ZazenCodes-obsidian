---
date: 2024-03-16
tags:
    - code-snip
hubs:
    - "[[python]]"
urls:
    - https://docs.pydantic.dev/latest/concepts/validators/
---

# Pydantic validation

Here's an example:

```python
from typing_extensions import Annotated
from pydantic import AfterValidator, BaseModel

VALID_PRIVACY_STATUSES = ("public", "private", "unlisted")


class YouTubeArgs(BaseModel):
    file: str
    title: str
    description: str
    category: str  # https://developers.google.com/youtube/v3/docs/videoCategories/list
    # see response example at bottom of this page
    keywords: str
    privacy_status: Annotated[
        str, AfterValidator(lambda x: x in VALID_PRIVACY_STATUSES)
    ]



```
