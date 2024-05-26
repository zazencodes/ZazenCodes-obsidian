---
date: 2024-03-05
tags:
    - fact
hubs:
    - "[[docker]]"
    - "[[devops]]"
urls:
    - https://github.com/Ayima/aura/blob/master/docker-compose.yaml
    - https://medium.com/@anasanjaria/code-reuse-in-docker-compose-using-yaml-anchor-feature-6cb2ff1d0427
---

# Docker common settings code reuse using yaml anchors

e.g. here's a production setup for a flask microservice `my_app`

```
version: "3"

x-common-settings:
  &common-settings
  env_file:
    - ./.env.dev
    # - ./.env.prod
  volumes:
    - "./src/api:/app"
    - "./logs:/logs"
    - "./script:/script"

services:
  redis:
    image: redis:7.0.6
    ports:
      - "6379:6379"
    volumes:
      - "./data/redis:/data"
  api:
    <<: *common-settings
    build: ./src/api
    depends_on:
      - "redis"
    restart: always
    # command: python manage.py run -h 0.0.0.0 -p 5000
    command: gunicorn --bind 0.0.0.0:5000 --workers 2 --timeout 600 manage:app
    expose:
      - 5000
  celery_worker:
    <<: *common-settings
    build: ./src/api
    depends_on:
      - "redis"
    command: celery --app my_app.adapters.celery.app worker --loglevel INFO
  flower:
    <<: *common-settings
    build: ./src/api
    depends_on:
      - "redis"
      - "celery_worker"
    ports:
      - 5555:5555
    command: /script/start_flower.sh
  nginx:
    build: ./src/nginx
    restart: always
    ports:
      - 80:80
    depends_on:
      - api
  daemon:
    <<: *common-settings
    image: alpine:3.18
    command: /script/daemon_tasks.sh


```
