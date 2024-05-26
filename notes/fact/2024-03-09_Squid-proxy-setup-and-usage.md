---
date: 2020-03-09
tags:
    - fact
hubs:
    - "[[linux]]"
urls:
    - https://linuxize.com/post/how-to-install-and-configure-squid-proxy-on-ubuntu-20-04/
---

# Squid proxy setup and usage

```bash
# Install squid
sudo apt update
sudo apt install squid

# Backup config
sudo cp /etc/squid/squid.conf{,.orginal}

# Add allowed IPs
touch /etc/squid/allowed_ips.txt

# Configure squid
vim /etc/squid/squid.conf
# (add lines like the ones below)
# ...
# acl allowed_ips src "/etc/squid/allowed_ips.txt"
# ...

#http_access allow localnet
http_access allow localhost
http_access allow allowed_ips

# And finally deny all other access to this proxy
http_access deny all

# Make requests from allowed hosts
python
>>> import requests
>>> PROXY_IP = "XXXXXXXXX"
>>> proxies = dict(http="http://{PROXY_IP}:3128/", https="http://{PROXY_IP}:3128/")
>>> resp = requests.get("https://gist.github.com/", proxies=proxies)
>>> resp # should be status code 200

# Access with chrome
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --user-data-dir="$HOME/proxy-profile" --proxy-server="http://XXXXXXXXX:3128"
```
