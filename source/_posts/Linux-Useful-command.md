---
title: 实用Linux命令整理
date: 2017-09-09 17:16:00
tags: Linux
---

#### 文件内容替换

``` bash
sudo sed -i 's/aaa/bbb/g' `grep -Rl aaa order_migrate_conf/`
```

#### 查找目录下的所有文件中是否含有某个字符串,并且只打印出文件名

``` bash
grep -Rl 1496628000000 order_migrate_conf/
```

#### 查看超大文件，vim 慎用

``` bash
less file.log
```


#### 超大文件从后往前查找关键词kind_pay

``` bash
tac file_path | grep kind_pay
```

less file_path, G(go to file end), /kind_pay + enter, N(search key word reversely)

#### 分类查看各种状态的TCP连接

``` bash
ss  -tan|awk 'NR>1{++S[$1]}END{for (a in S) print a,S[a]}'
```

#### 查看logs目录下所有文件夹及其内容的大小

``` bash
du -sh logs/*
```