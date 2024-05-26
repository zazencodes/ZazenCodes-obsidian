---
date: 2024-03-09
tags:
    - code-snip
hubs:
    - "[[python]]"
---

# Python logging

Setup logger
```python
import logging
logging.basicConfig(level=logging.INFO, format="%(asctime)s;%(levelname)s;%(message)s")
logger = logging.getLogger(Path().resolve().parent.name)
```

Setup logger with file handler for current file
```python
import os
os.environ["LOGGING_DIR"] = "/app/logs"
os.environ["LOGGING_FORMAT"] = "%(asctime)s;%(levelname)s;%(message)s"
os.environ["LOGGING_LEVEL"] = "INFO"

import logging
logger = logging.getLogger(__name__)
logging_fh = logging.FileHandler(
    Path(os.getenv("LOGGING_DIR", "/tmp")) / "logfile_id.log"
)
logging_fh.setFormatter(logging.Formatter(os.getenv("LOGGING_FORMAT")))
logging_fh.setLevel(os.getenv("LOGGING_LEVEL"))
logger.addHandler(logging_fh)
```



