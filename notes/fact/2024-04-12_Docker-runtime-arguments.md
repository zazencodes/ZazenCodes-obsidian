---
date: 2024-04-12
tags:
    - fact
hubs:
    - "[[docker]]"
    - "[[devops]]"
urls:
    - https://galea.medium.com/docker-runtime-arguments-604593479f45
---

# Docker runtime arguments

Below I run my_app (e.g. `my_app/__main__.py`) and pass the runtime argument `val1`

## Runtime arguments with CMD
```bash
# Dockerfile
FROM python:3.8-slim
WORKDIR /app
COPY . .

# Bash
docker build -t my-app .
docker run --rm my-app python -m my_app python -m my_app val1
```

## Runtime arguments with ENTRYPOINT
```bash
# Docker
FROM python:3.8-slim
WORKDIR /app
COPY . .
ENTRYPOINT ["python", "-m", "my_app"]
CMD ["default_value"]

# Bash
docker build -t my-app .
docker run --rm my-app val1
```

## Runtime arguments with ENV
```bash
# Docker
FROM python:3.8-slim
WORKDIR /app
COPY . .
ENV ARG_1=default_value
CMD ["python", "-m", "my_app"]

# Bash
docker build -t my-app .
docker run --rm -e ARG_1=val1 my-app
```

Or with compose

```bash
# Docker
version: "3.0"
services:
  my-app:
    build: .

# Bash
docker compose build my-app
docker compose run -e ART_1=val1 my-app
```

