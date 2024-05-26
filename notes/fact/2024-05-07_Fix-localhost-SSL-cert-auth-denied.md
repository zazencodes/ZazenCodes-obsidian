---
date: 2024-05-07
tags:
    - fact
hubs:
    - "[[web-dev]]"
---

# Fix localhost SSL cert auth denied

Getting error when attempting to log in to local web app at http://localhost:8080/login

e.g. in the console
```
auth.js:16 Error: Network Error
...
POST https://localhost/v1/auth/login net::ERR_CERT_COMMON_NAME_INVALID
```

Problem: Browser is blocking the request because it is using SSL

Solution: Go to https://localhost on your browser and then allow the connection (easy solution without e.g. needing to change configs)




```
