---
layout: post
title:  "Hosting Multiple Web Servers Behind A Proxy Server On A Debian LXC"
date:   2025-03-09 03:55:36 +1300
categories: linux guide install
---

### Create and configure the container

I used a Debian 12 LXC with 1 core, 512 MB RAM, 512 MB Swap and 8 G of disk space.

Update the container.
```bash
apt update && apt upgrade
```

---
### Configure the web servers

Create the root directory for each website.
```bash
mkdir -p /var/www/nastray.com
mkdir -p /var/www/test.nastray.com
mkdir -p /var/www/test.fabricatedfoundations.com
mkdir -p /var/www/fabricatedfoundations.com
```

Create a configuration file for each site.
```bash
nano /etc/nginx/sites-available/nastray.com
nano /etc/nginx/sites-available/test.nastray.com
nano /etc/nginx/sites-available/fabricatedfoundations.com
nano /etc/nginx/sites-available/test.fabricatedfoundations.com
```

Add the following to each file changing the server name and root for each site.
```bash
server {
    listen 80;
    listen [::]:80;

    server_name www.nastray.com nastray.com;

    root /var/www/nastray.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Create symbolic links to enable the sites.
```bash
ln -s /etc/nginx/sites-available/nastray.com /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/test.nastray.com /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/fabricatedfoundations.com /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/test.fabricatedfoundations.com /etc/nginx/sites-enabled/
```

Create a placeholder HTML file to identify each site.
```bash
echo "nastray.com" > /var/www/nastray.com/index.html
echo "test.nastray.com" > /var/www/test.nastray.com/index.html
echo "fabricatedfoundations.com" > /var/www/fabricatedfoundations.com/index.html
echo "test.fabricatedfoundations.com" > /var/www/test.fabricatedfoundations.com/index.html
```

Restart NGINX.
```bash
systemctl restart nginx
```

---
### Create and configure the container for the reverse proxy

I used a Debian 12 container again with the same resources as the first one.

Update the container.
```bash
apt update && apt upgrade
```

Create the configuration file for the reverse proxy.
```bash
nano /etc/nginx/sites-available/reverse-proxy
```

Copy and paste this into the configuration file, make necessary changes if needed.
```bash
server {
    listen 80;
    
    server_name nastray.com;

    location / {
        proxy_pass http://192.168.1.44:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;

    server_name test.nastray.com;

    location / {
        proxy_pass http://192.168.1.44:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;

    server_name fabricatedfoundations.com;

    location / {
        proxy_pass http://192.168.1.44:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

server {
    listen 80;

    server_name test.fabricatedfoundations.com;

    location / {
        proxy_pass http://192.168.1.44:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable the reverse proxy site.
```bash
ln -s /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites-enabled/
```

Reload NGINX
```bash
systemctl reload nginx
```

NOTES 
I enabled the firewall on Proxmox for the reverse proxy container allowing incoming traffic on ports 80 and 433, might have to enable some outgoing rules as well.

For the container running the web servers, I have not added any firewall or rules as of yet. I need to sort out how users will access the web root to build and manage their websites.
