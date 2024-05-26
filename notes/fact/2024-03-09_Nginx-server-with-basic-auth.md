---
date: 2022-03-09
tags:
    - fact
hubs:
    - "[[linux]]"
---

# Nginx server with basic auth

```bash
# install nginx
sudo apt-get update
sudo apt-get install nginx

# create passwords
echo -n 'user-name:' >> /etc/nginx/.htpasswd
openssl passwd -apr1 >> /etc/nginx/.htpasswd

# add auth_basic and auth_basic_user_file to location block:
# location / {
#   try_files $uri $uri/ =404;
#   auth_basic "Restricted Content";
#   auth_basic_user_file /etc/nginx/.htpasswd;
# }

# then restart after making any changes
sudo service nginx restart
```

