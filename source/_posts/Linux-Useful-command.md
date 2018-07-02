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
```
$ du -sh Movies/* | sort -nr
 28G	Movies/Pan's.Labyrinth.潘神的迷宫.2006.1080p.BluRay.REMUX.MPEG-4.AVC.PCM-WARHD
 19G	Movies/The.Lord.Of.The.Rings.The.Return.Of.The.King.2003.1080p.BluRay.x264-SiNNERS
 16G	Movies/The.Lord.Of.The.Rings.The.Fellowship.Of.The.Ring.2001.1080p.BluRay.x264-SiNNERS
 15G	Movies/Interstellar.星际穿越.2014.1080p.BluRay.x264.DTS-RARBG
 14G	Movies/Witness.for.the.Prosecution.控方证人.1957.1080p.BluRay.x264.DTS-WiKi
 14G	Movies/Se7en.七宗罪.1995.REMASTERED.1080p.BluRay.x264.DTS-ES-FGT
 14G	Movies/In.Bruges.杀手没有假期.2008.1080p.BluRay.x264.DTS-FGT
 13G	Movies/Saving.Private.Ryan.拯救大兵瑞恩.1998.1080p.BluRay.x264-LEVERAGE
 13G	Movies/Braveheart.勇敢的心.1995.1080p.BluRay.x264-CiNEFiLE
 12G	Movies/The.Hunt.狩猎.2012.DANISH.1080p.BluRay.x264.DTS-FGT
 12G	Movies/The.Dark.Knight.Rises.蝙蝠侠.黑暗骑士崛起.2012.1080p.BluRay.x264-ALLiANCE
 12G	Movies/Life.of.Pi.少年派的奇幻漂流.2012.1080p.BluRay.x264.DTS-FGT
 11G	Movies/The.Lost.City.of.Z.迷失Z城.2016.1080p.BluRay.x264-GECKOS[rarbg]
 11G	Movies/Manhattan.Murder.Mystery.曼哈顿谋杀疑案.1993.1080p.BluRay.X264-AMIABLE[rarbg]
 11G	Movies/Like.Father.Like.Son.如父如子.2013.1080p.BluRay.DTS.x264-PublicHD
```

#### 将需要交互的命令的结果重定向到文件中

``` bash
telnet zk_ip 2181 | tee -a  -i someFile

envi
```

#### 查看上下文切换

``` bash
nmon
```

#### 查看某个端口的应用程序PID
``` bash
$ lsof -i:9696
COMMAND    PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
kingshard 4994 eric   24u  IPv6 0x2dd5bc594d052b1f      0t0  TCP *:9696 (LISTEN)

```

#### 模拟网络抖动
http://blog.csdn.net/weiweicao0429/article/details/17578011
``` bash
# tc qdisc add dev eth0 root netem delay 100ms
# tc qdisc del dev eth0 root netem delay 100ms
# tc qdisc del dev eth0 root
```

#### 进程死亡
dmesg