---
date: 2024-10-23
tags:
    - fact
hubs:
    - "[[linux]]"
---

# Squid proxy server ubuntu setup

1. Update Your System and install squid

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install squid -y
```

2. Configure Squid

Squid's main configuration file is located at /etc/squid/squid.conf. We need to edit this file to configure Squid.

Squid by default listens on port 3128.

```bash
# Backup original config
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.backup
```

3. Allow a list of IPs

Create the file /etc/squid/whitelist.txt:

```bash
sudo vim /etc/squid/allowed_ips.txt
```

Add your IP addresses and labels, one per line:

```bash
382.391.392.2  # Jim
293.11.392.4   # Steve's home
```

2. Modify the Squid Configuration to Use the Whitelist

Open your Squid configuration file (/etc/squid/squid.conf)

```bash
sudo vim /etc/squid/squid.conf
```

Make the following additions to include the whitelist file.

```bash
# Define the ACL for the whitelisted IPs, using the file
acl allowed_ips src "/etc/squid/allowed_ips.txt"

# Allow access only to the whitelisted IPs
http_access allow allowed_ips

# Deny all other access
http_access deny all
```

3. Reset Squid
After configuring Squid, restart the service to apply the changes:

```bash
sudo systemctl restart squid
sudo systemctl enable squid
```

4. Configure firewall

```bash
sudo ufw allow 3128/tcp   # Adjust the port number if necessary
sudo ufw status
```


5. Proxy Caching Settings (Optional)
To configure caching (for improving response time and reducing bandwidth usage), add or modify the following in the Squid configuration file:

```bash
cache_mem 256 MB      # Amount of RAM to use for caching
maximum_object_size 50 MB  # Maximum object size to cache
cache_dir ufs /var/spool/squid 10000 16 256  # Disk cache settings
```

6. Python Example of Using the Proxy to Make a Request

Get the server IP

```bash
ip addr | grep eth0
```

To monitor traffic or troubleshoot issues, you can check the Squid access logs:

```bash
tail -f /var/log/squid/access.log
```

You can use the `requests` library in Python to make a request through the Squid proxy server. Here’s an example:

```python
import requests

# Insert proxy
proxy_ip = ""

proxy = {
    "http": f"http://{proxy_ip}:3128",
    "https": f"http://{proxy_ip}:3128",
}

response = requests.get("http://example.com", proxies=proxy)

print(response.text)
```



