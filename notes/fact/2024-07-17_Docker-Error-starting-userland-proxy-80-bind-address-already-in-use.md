---
date: 2024-07-17
tags:
    - fact
hubs:
    - "[[docker]]"
    - "[[networking]]"
    - "[[linux]]"
urls:
    - https://stackoverflow.com/questions/37971961/docker-error-bind-address-already-in-use
---

# Docker Error starting userland proxy 80 bind address already in use

I get this error from docker when I try to map a port from my docker app to my computer, and that port is in use

First, identify what is using that port on the machine, e.g. for port 80

```bash
lsof -i -P -n | grep 80
```

In my case I saw this

```bash
apache2    945            root    4u  IPv6  23268      0t0  TCP *:80 (LISTEN)
apache2    947        www-data    4u  IPv6  23268      0t0  TCP *:80 (LISTEN)
apache2    948        www-data    4u  IPv6  23268      0t0  TCP *:80 (LISTEN)

```

Which meant that I had an apache server running. This was happening automatically on boot, so I shut it down and disabled it

```bash
systemctl stop apache2
systemctl disable apache2
```

It could be that docker itself is causing the ports to be used

```bash
docker ps -a # What is running?
docker rm -fv $(docker ps -aq)  # Remove all containers
```
