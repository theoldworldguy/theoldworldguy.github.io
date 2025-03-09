---
layout: post
title:  "Setting Up Ssh Keys"
date:   2025-03-09 23:22:36 +1200
categories: linux guide
---

# Setting Up SSH Keys


```
ssh-keygen
```
Copy new pub key to remote host
```
ssh-copy-id user@remote
```
Change ssh configuration options in
```
sudo nano /etc/ssh/sshd_conf
```

Lots of keys, use a config file
```
nano .ssh/config
```
Add this
```
Host quickname
     HostName ip-or-domain
     User     username
     IdentityFile	~/.ssh/keyfile 
```
only need to issue this to login
```
ssh quickname 
```
