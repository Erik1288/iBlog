---
title: Cobar-Performance-Test-With-Sysbench
date: 2017-12-13 19:11:35
tags: Cobar
---

### centos

``` bash
yum install sysbench
```

lua目录 /usr/share/sysbench/

由于Cobar不支持SQL 92的表创建，所以需要修改/usr/share/sysbench/oltp_common.lua

如果需要查看帮助

则运行: sysbench [lua_path] help
比如: sysbench /usr/share/sysbench/oltp_insert.lua help

sysbench --mysql-host=mac \
--mysql-port=4000 \
--mysql-user=root \
--mysql-password= \
--mysql-db=eric \
--threads=4 \
--table-size=10 \
--auto_inc=off \
--tables=5 \
--rand-type=uniform \
--db-driver=mysql \
--time=6000000 --events=0 \
--percentile=99 \
--report-interval=2 \
/usr/local/Cellar/sysbench/1.0.11/share/sysbench/oltp_insert.lua \

#cobar_1

sysbench --mysql-host=10.1.21.148 \
--mysql-port=8066 \
--mysql-user=user \
--mysql-password='user' \
--mysql-db=user \
--threads=400 \
--table-size=10 \
--auto_inc=off \
--tables=5 \
--rand-type=uniform \
--db-driver=mysql \
--time=6000000 \
--events=0 \
--percentile=99 \
--report-interval=5 \
/usr/share/sysbench/oltp_insert.lua \


#cobar_2

sysbench --mysql-host=10.1.21.147 \
--mysql-port=8066 \
--mysql-user=user \
--mysql-password='user' \
--mysql-db=user \
--threads=400 \
--table-size=10 \
--auto_inc=off \
--tables=5 \
--rand-type=uniform \
--db-driver=mysql \
--time=6000000 --events=0 \
--percentile=99 \
--report-interval=5 \
/usr/share/sysbench/oltp_insert.lua \

### 测试报告
[root@mq001 ~]# sysbench --mysql-host=10.1.21.147 --mysql-port=8066 --mysql-user=user --mysql-password='user' --mysql-db=user --threads=400 --table-size=10 --auto_inc=off --tables=5 --rand-type=uniform --db-driver=mysql --time=60 --events=0 --percentile=99 --report-interval=5 /usr/share/sysbench/oltp_insert.lua run
sysbench 1.0.9 (using system LuaJIT 2.0.4)

Running the test with following options:
Number of threads: 400
Report intermediate results every 5 second(s)
Initializing random number generator from current time


Initializing worker threads...

Threads started!

[ 5s ] thds: 400 tps: 11516.37 qps: 11516.37 (r/w/o: 0.00/11516.37/0.00) lat (ms,99%): 204.11 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 400 tps: 2884.87 qps: 2884.87 (r/w/o: 0.00/2884.87/0.00) lat (ms,99%): 831.46 err/s: 0.00 reconn/s: 0.00
[ 15s ] thds: 400 tps: 4025.00 qps: 4025.00 (r/w/o: 0.00/4025.00/0.00) lat (ms,99%): 1869.60 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 400 tps: 13497.32 qps: 13497.32 (r/w/o: 0.00/13497.32/0.00) lat (ms,99%): 142.39 err/s: 0.00 reconn/s: 0.00
[ 25s ] thds: 400 tps: 10908.17 qps: 10908.17 (r/w/o: 0.00/10908.17/0.00) lat (ms,99%): 144.97 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 400 tps: 12280.74 qps: 12280.74 (r/w/o: 0.00/12280.74/0.00) lat (ms,99%): 132.49 err/s: 0.00 reconn/s: 0.00
[ 35s ] thds: 400 tps: 9783.37 qps: 9783.37 (r/w/o: 0.00/9783.37/0.00) lat (ms,99%): 150.29 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 400 tps: 11703.52 qps: 11703.72 (r/w/o: 0.00/11703.72/0.00) lat (ms,99%): 173.58 err/s: 0.00 reconn/s: 0.00
[ 45s ] thds: 400 tps: 8253.06 qps: 8252.86 (r/w/o: 0.00/8252.86/0.00) lat (ms,99%): 207.82 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 400 tps: 7148.98 qps: 7149.18 (r/w/o: 0.00/7149.18/0.00) lat (ms,99%): 657.93 err/s: 0.00 reconn/s: 0.00
[ 55s ] thds: 400 tps: 10911.35 qps: 10911.15 (r/w/o: 0.00/10911.15/0.00) lat (ms,99%): 176.73 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 400 tps: 8702.48 qps: 8702.48 (r/w/o: 0.00/8702.48/0.00) lat (ms,99%): 235.74 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            0
        write:                           558494
        other:                           0
        total:                           558494
    transactions:                        558494 (9273.20 per sec.)
    queries:                             558494 (9273.20 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.2244s
    total number of events:              558494

Latency (ms):
         min:                                  3.89
         avg:                                 43.02
         max:                               2881.01
         99th percentile:                    253.35
         sum:                            24023700.41

Threads fairness:
    events (avg/stddev):           1396.2350/30.84
    execution time (avg/stddev):   60.0593/0.05



[root@mq001 ~]# sysbench --mysql-host=10.1.21.148 --mysql-port=8066 --mysql-user=user --mysql-password='user' --mysql-db=user --threads=400 --table-size=10 --auto_inc=off --tables=5 --rand-type=uniform --db-driver=mysql --time=60 --events=0 --percentile=99 --report-interval=5 /usr/share/sysbench/oltp_insert.lua run
sysbench 1.0.9 (using system LuaJIT 2.0.4)

Running the test with following options:
Number of threads: 400
Report intermediate results every 5 second(s)
Initializing random number generator from current time


Initializing worker threads...

Threads started!

[ 5s ] thds: 400 tps: 10470.98 qps: 10470.98 (r/w/o: 0.00/10470.98/0.00) lat (ms,99%): 262.64 err/s: 0.00 reconn/s: 0.00
[ 10s ] thds: 400 tps: 3020.06 qps: 3020.06 (r/w/o: 0.00/3020.06/0.00) lat (ms,99%): 1050.76 err/s: 0.00 reconn/s: 0.00
[ 15s ] thds: 400 tps: 1107.64 qps: 1107.64 (r/w/o: 0.00/1107.64/0.00) lat (ms,99%): 1327.91 err/s: 0.00 reconn/s: 0.00
[ 20s ] thds: 400 tps: 7395.23 qps: 7395.23 (r/w/o: 0.00/7395.23/0.00) lat (ms,99%): 816.63 err/s: 0.00 reconn/s: 0.00
[ 25s ] thds: 400 tps: 12654.11 qps: 12654.11 (r/w/o: 0.00/12654.11/0.00) lat (ms,99%): 125.52 err/s: 0.00 reconn/s: 0.00
[ 30s ] thds: 400 tps: 10854.72 qps: 10854.92 (r/w/o: 0.00/10854.92/0.00) lat (ms,99%): 153.02 err/s: 0.00 reconn/s: 0.00
[ 35s ] thds: 400 tps: 11832.81 qps: 11832.61 (r/w/o: 0.00/11832.61/0.00) lat (ms,99%): 186.54 err/s: 0.00 reconn/s: 0.00
[ 40s ] thds: 400 tps: 6895.44 qps: 6895.44 (r/w/o: 0.00/6895.44/0.00) lat (ms,99%): 257.95 err/s: 0.00 reconn/s: 0.00
[ 45s ] thds: 400 tps: 12907.06 qps: 12907.06 (r/w/o: 0.00/12907.06/0.00) lat (ms,99%): 158.63 err/s: 0.00 reconn/s: 0.00
[ 50s ] thds: 400 tps: 7760.80 qps: 7760.80 (r/w/o: 0.00/7760.80/0.00) lat (ms,99%): 292.60 err/s: 0.00 reconn/s: 0.00
[ 55s ] thds: 400 tps: 11595.84 qps: 11595.84 (r/w/o: 0.00/11595.84/0.00) lat (ms,99%): 161.51 err/s: 0.00 reconn/s: 0.00
[ 60s ] thds: 400 tps: 10314.36 qps: 10314.36 (r/w/o: 0.00/10314.36/0.00) lat (ms,99%): 142.39 err/s: 0.00 reconn/s: 0.00
SQL statistics:
    queries performed:
        read:                            0
        write:                           534637
        other:                           0
        total:                           534637
    transactions:                        534637 (8890.97 per sec.)
    queries:                             534637 (8890.97 per sec.)
    ignored errors:                      0      (0.00 per sec.)
    reconnects:                          0      (0.00 per sec.)

General statistics:
    total time:                          60.1304s
    total number of events:              534637

Latency (ms):
         min:                                  4.14
         avg:                                 44.91
         max:                               3066.99
         99th percentile:                    292.60
         sum:                            24012284.54

Threads fairness:
    events (avg/stddev):           1336.5925/23.11
    execution time (avg/stddev):   60.0307/0.03