---
date: 2024-05-20
tags:
    - fact
hubs:
    - "[[linux]]"
    - "[[macos]]"
---

# Installing binaries on Unix systems

Example with minikube on MacOS (Linux works same way)

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-arm64
sudo install minikube-darwin-arm64 /usr/local/bin/minikube
```

1. Download binary. `-L` flag follows redirects and `-O` flag saves file with same name as it has on the server.
2. Install command copies binary into `/usr/local/bin`, which is already in my PATH, and sets the proper executable permissions (`chmod +x`)



