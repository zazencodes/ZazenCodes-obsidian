---
date: 2023-09-08
tags:
    - blog
hubs:
    - "[[linux]]"
    - "[[devops]]"
urls:
    - https://hackernoon.com/micro-devops-with-systemd-supercharge-any-ordinary-linux-server
---

# Micro DevOps with systemd


For example, consider this simple `.service` that defines how to run a simple web service:

```bash
[Unit]
Description=a simple web service

[Service]
ExecStart=/usr/bin/python3 -m http.server 8080
```

When placed into a file like `/etc/systemd/system/webapp.service`, this creates a service that you can control with `systemctl`:


- `systemctl start webapp` will start the process.
- `systemctl status webapp` will display whether the service is running, its uptime, and output from `stderr` and `stdout` , as well as the process’s ID and other information.
- `systemctl stop webapp` will end the service.

to expose an API key to a `.service` unit, create a file at `/etc/credstore/api-key` to persist the file across reboots or at `/run/credstore/api-key` to avoid persisting the file permanently (the path can be arbitrary, but systemd will treat these `credstore` paths as defaults). In either case, the file should have restricted permissions using a command like `chmod 400 /etc/credstore/api-key`.



Under the `[Service]` section of the `.service` file, define the `LoadCredential=` option and pass it two values separated by a colon (`:`): the name of the credential and its path. For example, to call our `/etc/credstore/api-key` file “token,” define the following systemd service option:



```bash
LoadCredential=token:/etc/credstore/api-key
```



When systemd starts your service, the secret is exposed to the running service under a path of the form `${CREDENTIALS_DIRECTORY}/token` where `${CREDENTIALS_DIRECTORY}` is an environment variable populated by systemd. Your application code should read in each secret defined this way for use in libraries or code that require secure values like API tokens or passwords. For example, in Python, you can read this secret with code like the following:



```python
from os import environ
from pathlib import Path

credentials_dir = Path(environ["CREDENTIALS_DIRECTORY"])
with Path(credentials_dir / "token").open() as f:
    secret = f.read().strip()
```

