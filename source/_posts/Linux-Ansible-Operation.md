---
title: Linux-Ansible-Operation
date: 2018-07-25 19:34:47
tags:
---


sudo ansible -i ./zerodb-ansible/hosts proxy_public  -m "shell" -a "free -h" --private-key=~/.ssh/id_rsa -u nanxing

### hosts
[proxy_public]
10.12.0.241
10.12.1.[61:62]