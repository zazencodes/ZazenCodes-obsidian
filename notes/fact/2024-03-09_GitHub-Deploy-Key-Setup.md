---
date: 2020-06-24
tags:
    - fact
hubs:
    - "[[git]]"
    - "[[linux]]"
---

# GitHub Deploy Key Setup

1. Generate key
```
ssh-keygen
...
cp ./key-name* ~/.ssh
```

2. Go to GitHub repo and add key-name.pub

3. Clone repo using trick from [this stackoverflow thread](https://stackoverflow.com/a/4565746/3511819)
```
ssh-agent bash -c 'ssh-add ~/.ssh/key-name; git clone git@github.com:user/proj-name.git'
```

4. Open `~/.ssh/config` and add a block like this:
```
Host github-proj-name
HostName github.com
IdentityFile ~/.ssh/key-name
```

5. Finally, open `.git/config` in your new repo and change the hostname in the git connection string like this:
```
git@github-proj-name:user/proj-name.git
```

Now you will be able to push/pull without password authentication.

