---
title: Linux-Disk-io-test
date: 2018-09-22 14:20:03
tags:
---


### 测试写
time dd if=/dev/zero of=test.dbf bs=8k count=300000

### 测试读
time dd if=/dev/sda1 of=/dev/null bs=8k count=300000
