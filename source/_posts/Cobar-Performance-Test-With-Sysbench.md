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
--mysql-port=9696 \
--mysql-user=zerodb \
--mysql-password=zerodb \
--mysql-db=zerodb \
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
    
    
    
    

sysbench --mysql-host=10.1.134.195 \
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


sysbench --mysql-host=10.1.21.242 \
--mysql-port=9696 \
--mysql-user=zerodb \
--mysql-password='zerodb' \
--mysql-db=zerodb \
--threads=10 \
--debug=on \
--verbosity=5 \
--tables=5 \
--events=1 \
--auto_inc=off \
--table-size=1000 \
--db-driver=mysql \
--time=6000000 \
--percentile=99 \
--report-interval=5 \
/usr/share/sysbench/oltp_read_only.lua \
run



# zerodb
sysbench --mysql-host=10.1.21.242 \
--mysql-port=9696 \
--mysql-user=zerodb \
--mysql-password='zerodb' \
--mysql-db=zerodb \
--threads=10 \
--table-size=10 \
--auto_inc=off \
--tables=5 \
--rand-type=uniform \
--db-driver=mysql \
--time=6000000 --events=0 \
--percentile=99 \
--report-interval=5 \
/usr/share/sysbench/oltp_insert.lua \
cleanup

sysbench --mysql-host=10.1.21.242 \
--mysql-port=9696 \
--mysql-user=zerodb \
--mysql-password='zerodb' \
--mysql-db=zerodb \
--threads=10 \
--table-size=10 \
--auto_inc=off \
--tables=5 \
--rand-type=uniform \
--db-driver=mysql \
--time=6000000 --events=0 \
--percentile=99 \
--report-interval=5 \
/usr/share/sysbench/oltp_insert.lua \
prepare

sysbench --mysql-host=10.1.21.242 \
--mysql-port=9696 \
--mysql-user=zerodb \
--mysql-password='zerodb' \
--mysql-db=zerodb \
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
run

WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[899:{SchemaIndex=225, TableIndex=899, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (1592131459, 7, '55016285732-53347045338-85834558147-71206675947-20854212178-67704861318-55023920639-15021922692-58783263226-91831209826', '44804973802-80594424819-30662288113-07543047794-69158617276')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 3, mergeExecResult: 0. shardingNode:map[607:{SchemaIndex=152, TableIndex=607, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (1551461983, 1, '97565032345-06095273461-70708868939-89114711497-07191112359-84320966593-06915637429-34011520393-29123922986-41181266499', '10350064023-47161933420-32840507920-65997397770-16177253238')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 1000, mergeExecResult: 0. shardingNode:map[371:{SchemaIndex=93, TableIndex=371, HostGroup=hostGroup2}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (1688397171, 3, '66563027950-07665251035-46015031422-61663761332-16063948763-45273359443-68411883367-29428347771-65572934442-17492196263', '60851935379-49894017653-67952828882-67412344041-59230452535')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[1011:{SchemaIndex=253, TableIndex=1011, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (-573803507, 2, '43589791346-92222019297-77586384271-52778612091-96284921979-05035493725-49350672130-03697130813-21052521840-31158762545', '33228172883-24963516238-59142423481-48021525640-21772792414')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 1001, mergeExecResult: 0. shardingNode:map[310:{SchemaIndex=78, TableIndex=310, HostGroup=hostGroup2}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (439335222, 7, '03793368943-50266953542-49577833032-14468376728-10198261385-85128057368-15281808692-43095784178-01078624681-09027567677', '45982088127-90369291605-80044281219-46887615792-75467747528')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 999, mergeExecResult: 0. shardingNode:map[259:{SchemaIndex=65, TableIndex=259, HostGroup=hostGroup2}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (824478979, 9, '79929455805-82686000395-87933678180-97427805059-78850675331-66749630290-23371965370-59789760435-11687903028-37111392097', '65387439360-90889047465-34899315444-83593158053-88419722270')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 1000, mergeExecResult: 0. shardingNode:map[388:{SchemaIndex=98, TableIndex=388, HostGroup=hostGroup2}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (1778834820, 4, '26400620948-86886163258-66955677020-24736956201-37633810654-62170837016-10953232786-08731106510-27930064178-11956848338', '22734010461-63969213016-54927682118-22921617753-25869637636')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 1000, mergeExecResult: 0. shardingNode:map[345:{SchemaIndex=87, TableIndex=345, HostGroup=hostGroup2}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-1906262361, 4, '73054929345-92331883054-27511894619-20000378864-11948567194-86713567705-11190297445-34047180586-99650690403-71903892017', '07335571244-33555087386-76380202973-29490934882-40814481851')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[569:{SchemaIndex=143, TableIndex=569, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (1606555193, 2, '94281857928-97538212927-39261666932-79938992409-52558016112-06785833668-50810169070-34555719774-82330250972-35853785503', '29683325026-64124354800-63566774522-19722425169-37547560762')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[906:{SchemaIndex=227, TableIndex=906, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (1143670666, 7, '74870370257-63163303467-80061015187-84192128665-25175084977-80554698357-24269465921-53186773519-78675020561-83488552374', '73368269204-64304332325-34620567277-68846684629-38045820340')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[941:{SchemaIndex=236, TableIndex=941, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (-1370776493, 6, '16692170437-43512178684-59516594203-75377838995-76081749306-71167865747-97237301685-23453441670-03667446181-56604717727', '14195711604-53513247591-07873804194-96431257292-05520461785')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[872:{SchemaIndex=219, TableIndex=872, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (-1208557416, 5, '27703568859-81266825379-86789890837-60513053664-22258866897-34966682979-82761864561-83949706302-60135012966-31647733109', '63271991640-87709697591-13072925018-38646644832-79179029573')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[991:{SchemaIndex=248, TableIndex=991, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (138564575, 6, '26857888685-88370802193-51045899118-07572056078-70111763976-05806767425-30208609378-20016792779-48308493020-29821526191', '61555808494-05287717368-66884245341-45983655335-60458704606')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 1007, mergeExecResult: 0. shardingNode:map[449:{SchemaIndex=113, TableIndex=449, HostGroup=hostGroup2}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-229865921, 8, '14034106238-21876701329-38233326181-31158038417-73928237004-22908354046-72713486563-40610785127-99166317018-52234063725', '23168975405-73598149943-63125122764-74527985990-06171559404')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[959:{SchemaIndex=240, TableIndex=959, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-677583807, 4, '40735866383-98457016615-08789114417-03599576561-48782283253-32055567198-88072330194-83020475968-03693883045-96278614943', '70150968961-19151317381-16983120036-88185867422-84862477300')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 1006, mergeExecResult: 0. shardingNode:map[406:{SchemaIndex=102, TableIndex=406, HostGroup=hostGroup2}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-2093701526, 5, '09210536591-41896791043-73295525884-89613310006-75387609760-41655764025-47137540812-97100201238-15965683915-13727887851', '48024514833-13441113380-49081922507-17183150276-06208568092')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[894:{SchemaIndex=224, TableIndex=894, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (108403582, 4, '58477996001-39515710854-83438883003-74672331816-19403553257-36113016280-00822951895-43706580851-50122510142-72629874469', '25950012615-74047249694-75644797724-67405314152-48742553448')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[824:{SchemaIndex=207, TableIndex=824, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (1108509496, 2, '05591703103-07428761202-09014510822-44571443153-87746638001-18005366596-37591757698-85345493107-52688524466-97341165803', '90287453968-47714845240-54141462203-72968014933-11681199861')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[960:{SchemaIndex=241, TableIndex=960, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (-432972736, 4, '32013395637-31037790439-99508763633-39582502523-46934970326-99846192559-79844868065-81128826635-19475623448-94555401729', '72848617704-77451404798-55770002912-34209086424-23821979141')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[864:{SchemaIndex=217, TableIndex=864, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (209038176, 6, '31725269303-67222682770-97738396828-19717571752-01126165612-57542830563-51718817223-13687491781-38891241679-10677758794', '40604117998-67635443824-03882862739-68433432919-44278589057')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[525:{SchemaIndex=132, TableIndex=525, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-663100941, 3, '13572309986-10356575142-87428874956-18162116973-83396135908-09352768769-83603227505-94165603992-17437041978-81761343722', '58884262253-26008842705-81702669160-02709154611-78739667686')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 3, mergeExecResult: 0. shardingNode:map[821:{SchemaIndex=206, TableIndex=821, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-1290297141, 6, '28650953615-66926208686-47052429218-34545146184-51693579732-60563146329-00350338711-30136625347-99506216978-22404723673', '30345581431-18594298965-23203468098-48853746952-30033247101')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[881:{SchemaIndex=221, TableIndex=881, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (2128759665, 3, '23557322059-62351008582-19218938648-55023996016-67978090662-04498693910-40333149725-65457542876-11556698223-86845640120', '39531266486-08252986785-52805405505-62972899514-62765043827')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[829:{SchemaIndex=208, TableIndex=829, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-1081064253, 2, '39396011527-03550826801-53544579241-44145722341-56423193708-85298116954-45172197891-54532848324-82241582599-97563898680', '76749800295-63255838342-73258070193-02305060704-64352300433')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[900:{SchemaIndex=226, TableIndex=900, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-513806212, 1, '16727493623-40348083665-41922314110-05150944505-43353212884-59194917800-00753396008-63562258886-84259052626-93713085152', '38634951090-07591809914-73325301815-73315254313-82978384844')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 1, mergeExecResult: 0. shardingNode:map[863:{SchemaIndex=216, TableIndex=863, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-450094943, 6, '23924438266-62111362125-81473873200-06328498397-88922636306-39182947784-05124699385-49780077333-57313760145-54634535492', '96577223660-83042935121-28454288566-32057418883-39061132652')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 3, mergeExecResult: 0. shardingNode:map[963:{SchemaIndex=241, TableIndex=963, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (1800163267, 4, '27948346446-05701199368-49149768636-18762164887-01870692332-13661773677-35851324771-46733712098-36653074427-31181566769', '08781105428-22812332073-21224324603-72676647429-86438534524')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[878:{SchemaIndex=220, TableIndex=878, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-1535190894, 9, '46310600095-97541020554-94065740469-24853513335-69630566641-31721053084-60343664925-46348792383-87749652424-52899212634', '41944783155-15764748316-26823908956-07652542569-85074827204')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[906:{SchemaIndex=227, TableIndex=906, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-1945995146, 4, '64447284661-78831878306-27636005022-88863833954-72613803410-72661880191-81775646793-56738842216-16376642732-30662007847', '44349800594-98690127361-78211321236-38340738575-94555021426')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[907:{SchemaIndex=227, TableIndex=907, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (-1719256971, 3, '36693810506-14727560109-48871542333-86332816255-91899190554-43809786563-22813456576-32468291237-08105779091-35654765264', '76792216525-13152293559-11187746420-35413475429-73316527358')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 3, mergeExecResult: 0. shardingNode:map[827:{SchemaIndex=207, TableIndex=827, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (-1191957307, 3, '02373892720-70783246981-33072353568-68101913427-25982074179-25716557733-53487403333-33176741697-19961659954-24745733546', '65228452942-44586411798-63293957289-46889416058-55247585232')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[715:{SchemaIndex=179, TableIndex=715, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (-368940747, 10, '83732843347-38915816227-71028864502-83867874900-55259635749-04867892981-61246311166-86657029985-98371125854-01109713202', '28665022675-32159696841-27962719413-92137483719-65438251505')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 3, mergeExecResult: 0. shardingNode:map[531:{SchemaIndex=133, TableIndex=531, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (296709651, 4, '68512385573-12594713777-31891099071-55341413329-96572532552-89441743744-75535604911-39664974035-50293453830-41657909966', '54516090919-10379612997-70745484084-47657473576-34684395884')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 2, mergeExecResult: 0. shardingNode:map[1010:{SchemaIndex=253, TableIndex=1010, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (375408626, 5, '86917579879-69750606525-04310713441-66206378609-22671203503-85955817483-08099675016-87194148638-56030296183-72576969654', '14638356398-42816005551-86372571322-85471822123-78848498639')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 207, mergeExecResult: 0. shardingNode:map[481:{SchemaIndex=121, TableIndex=481, HostGroup=hostGroup2}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-453506529, 7, '05922416371-88423926572-06168309932-36230491079-35838894643-96519731061-10785734640-51619035736-36739664574-07173836937', '86251074060-14148969607-50675271708-72280786641-89845672651')] 
WARN[0449] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 999, mergeExecResult: 0. shardingNode:map[506:{SchemaIndex=127, TableIndex=506, HostGroup=hostGroup2}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (923516410, 1, '71118741783-11388565696-96706199622-38503412573-05309353257-45864148538-09400667746-21110528894-17837136283-91361843504', '67292585002-85708710331-29800085937-37291514623-47320174130')] 




WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 306, mergeExecResult: 0. shardingNode:map[540:{SchemaIndex=136, TableIndex=540, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (1344100892, 4, '37923605607-42327774961-35128132820-48158088043-25957128013-51759124002-40449799093-68433809807-31405796686-22817252999', '94291235269-60160247565-39950122186-97684115599-96232437155')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 211, mergeExecResult: 0. shardingNode:map[540:{SchemaIndex=136, TableIndex=540, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-757833244, 5, '50104262688-83666007823-06989738144-95141595109-93329094811-13832991101-93076033912-05017862363-35692005246-88744289137', '54081433849-09378248558-38274752100-88855316211-27289637272')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 3, mergeExecResult: 0. shardingNode:map[198:{SchemaIndex=50, TableIndex=198, HostGroup=hostGroup1}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (652469446, 1, '07652454524-22981957011-41438122391-90347773905-88663504044-48425601588-32127975840-28199749965-27298300589-25715051794', '60470366939-49887579866-34848349508-29262200362-52000338086')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 223, mergeExecResult: 0. shardingNode:map[670:{SchemaIndex=168, TableIndex=670, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-212205214, 3, '39093408635-10213704095-82878149257-88673860178-07278582464-52697390774-95388237318-03462450231-66036638124-64228596635', '38716722580-86486363760-61985600779-83841271441-42765235992')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 248, mergeExecResult: 0. shardingNode:map[594:{SchemaIndex=149, TableIndex=594, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (-1136835154, 2, '23757270379-51595389012-47476784586-71294005118-20959815636-40533304860-41036462462-22519409530-16151251756-29363403201', '02895678318-89791495824-44276206517-37575326693-35714861579')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 196, mergeExecResult: 0. shardingNode:map[596:{SchemaIndex=150, TableIndex=596, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (927250004, 3, '02232667199-83769701794-18129066725-31520472849-93375763771-42931518091-56094856485-24182688367-31807939112-07474880757', '46530309679-12705910558-69428237706-14629411089-19801742204')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 213, mergeExecResult: 0. shardingNode:map[568:{SchemaIndex=143, TableIndex=568, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (-1908502072, 6, '11131352749-42538464007-16895664672-62644300750-86974265339-82077852146-68660984992-78157770948-82596046151-02353070903', '35289675345-82804575493-00647347117-53443940605-48670213485')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 223, mergeExecResult: 0. shardingNode:map[530:{SchemaIndex=133, TableIndex=530, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (1565462034, 2, '03766551315-88042959333-06619558863-52259377137-27072539179-52210031586-63547684874-18650692849-22136096911-63673950864', '23300762166-38283070103-67223042934-76314148059-35039279078')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 252, mergeExecResult: 0. shardingNode:map[705:{SchemaIndex=177, TableIndex=705, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-282929857, 9, '78018924775-94910259974-14900983402-30428448745-60064721591-40668444882-53712901782-65346911189-59688209606-74312372904', '82234943021-50353143509-80352130420-57571772487-81475633439')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 291, mergeExecResult: 0. shardingNode:map[724:{SchemaIndex=182, TableIndex=724, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (2109119188, 7, '36086413796-51767176683-52982238848-36460965892-84578503998-21697029291-39139590496-18305440259-61532992900-70589028518', '30048526034-08924322516-87289516951-51922079500-47125778015')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 210, mergeExecResult: 0. shardingNode:map[573:{SchemaIndex=144, TableIndex=573, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (-994558525, 8, '74078055913-58294585862-48131723756-93089582597-72949713745-85429496032-27905841748-82110966829-12174780172-45947256767', '52598986037-80854622196-65516850169-26314282473-46184813701')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 99, mergeExecResult: 0. shardingNode:map[701:{SchemaIndex=176, TableIndex=701, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (1290611389, 10, '06010961323-54786660073-11823817491-71384866194-20002003501-13527828508-52664186690-95551041252-11585296099-86883272955', '35567244596-82663971593-11831798779-27870930145-87441039863')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 292, mergeExecResult: 0. shardingNode:map[567:{SchemaIndex=142, TableIndex=567, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (810312247, 10, '41210873160-21444867641-91122365510-19036499914-34569882163-29091701055-84873343605-60726600041-00757500001-24263691698', '41652074477-40057441865-33409084588-16838407116-03941310610')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 3, mergeExecResult: 0. shardingNode:map[213:{SchemaIndex=54, TableIndex=213, HostGroup=hostGroup1}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (1881853141, 2, '14742564348-19913327041-96137703234-33860693319-12765733015-19685217285-90407570147-69715707192-71609875203-72224693707', '62398998246-64288014470-28245239723-29976129053-08238346550')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 242, mergeExecResult: 0. shardingNode:map[698:{SchemaIndex=175, TableIndex=698, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (1011920570, 2, '33865708347-82274332444-45097600411-92274439334-16926356904-33828206076-16463987600-06687756787-06453656924-53622006994', '48462480629-04943720987-66250435599-66843177447-86484400166')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 195, mergeExecResult: 0. shardingNode:map[710:{SchemaIndex=178, TableIndex=710, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (-2128817862, 7, '12483300987-88535524922-66334741529-64659385522-75032596868-93466219012-01222978665-55122702202-84142740410-53583589563', '89132580254-79206347601-13534456013-96027776935-07650599065')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 233, mergeExecResult: 0. shardingNode:map[544:{SchemaIndex=137, TableIndex=544, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (1496016416, 9, '99154323337-90872497215-53698284897-04686362817-35622767062-48356019022-04016504226-75493501145-38429448892-66989460733', '84409926268-28404056573-96777093808-13481662538-98898905310')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 27, mergeExecResult: 0. shardingNode:map[907:{SchemaIndex=227, TableIndex=907, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (756613003, 1, '48041615531-14651008388-14559884387-33045819966-91038786346-39394412181-13147015451-43265025586-90014901196-20159726596', '62496325099-63432555894-84244639858-58924561742-12138512609')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 284, mergeExecResult: 0. shardingNode:map[525:{SchemaIndex=132, TableIndex=525, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (588268045, 3, '01778869912-44391681202-43730989451-99650497628-58071023040-36244500937-05173857724-21592385062-52662008534-88410130321', '68551597952-73268487812-26478068307-11227961732-49092247276')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 229, mergeExecResult: 0. shardingNode:map[674:{SchemaIndex=169, TableIndex=674, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (75111074, 1, '07857287095-83026714634-10012972058-11441297287-80787536524-31038348780-79408482259-95286783641-23620583564-94195135059', '39714908056-38058099094-71552841772-10457961132-39419941227')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 230, mergeExecResult: 0. shardingNode:map[577:{SchemaIndex=145, TableIndex=577, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-911965761, 8, '07321418565-94055920833-16462957068-14933742620-00134077423-05303377934-66121817102-49230219319-72774150061-80468601177', '26522648558-75134929821-83594530381-47917024515-31668467194')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 232, mergeExecResult: 0. shardingNode:map[513:{SchemaIndex=129, TableIndex=513, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (1315070465, 5, '15474433240-84784850443-17796367719-48021520867-10943303644-48200729918-91013947900-33061132253-28956163141-98846516045', '99644114596-46918915095-13462299586-15204837308-12785249281')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 231, mergeExecResult: 0. shardingNode:map[737:{SchemaIndex=185, TableIndex=737, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (-180994785, 9, '61681162669-07252910162-88941330016-07633567093-86234290556-74605484081-92189953463-28499401861-08881094341-04206273246', '73069152681-47514577462-53755512339-63165295325-04957863379')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 225, mergeExecResult: 0. shardingNode:map[583:{SchemaIndex=146, TableIndex=583, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (905672263, 5, '11686869034-48032081508-20645314791-73241768758-13634493369-36096577158-76040275597-65556337906-73745369125-93817830861', '25154935190-35760055237-67095134841-73686707222-08327146392')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 227, mergeExecResult: 0. shardingNode:map[568:{SchemaIndex=143, TableIndex=568, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (1028081208, 6, '65787201754-47681590360-50036937953-38544197509-01589612011-17576022949-32999165698-83171817746-37315560344-14463882405', '65637926145-34138639858-41094177281-29428779399-44688752051')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 232, mergeExecResult: 0. shardingNode:map[558:{SchemaIndex=140, TableIndex=558, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (278584878, 10, '32358945918-66413584434-88058020708-48390261093-67208582401-88568979078-90497400157-49426845556-80952580018-13610087462', '14198786100-85805610838-89824761297-62058901344-60492560838')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 228, mergeExecResult: 0. shardingNode:map[649:{SchemaIndex=163, TableIndex=649, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (1183431305, 2, '43858458237-46788126262-05213368147-07302202409-13593899883-99375055319-42875375550-56499216168-01796902390-08647652700', '48205424174-34611724289-69246697079-03898745311-99595319598')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 26, mergeExecResult: 0. shardingNode:map[1002:{SchemaIndex=251, TableIndex=1002, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-1701967850, 9, '59147040383-79705593723-62886101805-60535881029-18413556495-06238443919-02766097021-38361379870-67179305438-52530038201', '14770296700-13396847549-83905253251-99337969432-58669912709')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 229, mergeExecResult: 0. shardingNode:map[745:{SchemaIndex=187, TableIndex=745, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (-1169184489, 7, '64955629118-36147506634-51676575477-92147977264-45999410094-15567300060-12836575112-28470342888-04361926080-90629622527', '86593300354-87140619605-87718697448-88392415837-04795141111')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 272, mergeExecResult: 0. shardingNode:map[662:{SchemaIndex=166, TableIndex=662, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (1555424918, 10, '11260581771-57660361810-58723152189-43635886031-41094065361-37903459575-65856768363-01843670891-17848285384-92171598763', '36462031678-64770373669-77316073133-32672501580-50753545368')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 271, mergeExecResult: 0. shardingNode:map[736:{SchemaIndex=185, TableIndex=736, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (-270642912, 9, '87806093665-04774113218-83983577041-09351362099-88339377062-22401969710-89370302094-61598971830-04573362584-60421137580', '26415700902-09148763904-87375314444-00285784497-80449574080')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 222, mergeExecResult: 0. shardingNode:map[588:{SchemaIndex=148, TableIndex=588, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (-1608970828, 2, '75571666926-33738024136-84313793696-10714487345-57088294174-41711334397-51382524640-08327299973-66970622008-15344703881', '73366501022-14368954292-21130701303-33960859630-28160972955')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 220, mergeExecResult: 0. shardingNode:map[556:{SchemaIndex=140, TableIndex=556, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (-993863212, 3, '68136313990-05892185009-94544559127-32267303237-37763842514-89959134082-97215598641-72307869276-49642816905-24994127953', '97111382859-69888150629-89398585714-99569889618-30099083026')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 220, mergeExecResult: 0. shardingNode:map[513:{SchemaIndex=129, TableIndex=513, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-1299240449, 6, '13580427730-91599560017-42392676834-62860979245-65274043134-68243135520-42672727926-56794775679-69217526806-27227304772', '15736027942-38175115801-93773367279-77944721463-03173899187')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 214, mergeExecResult: 0. shardingNode:map[579:{SchemaIndex=145, TableIndex=579, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-1346301507, 2, '95777728263-97322915970-24025444526-11872987788-74589515383-37108287704-62890589663-84327075019-58161194629-66166453693', '43925335448-41493749427-45817169562-78879091928-39249185154')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 252, mergeExecResult: 0. shardingNode:map[723:{SchemaIndex=181, TableIndex=723, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest3 (id, k, c, pad) VALUES (-1604556499, 3, '54964824319-34611710514-15902154995-18014105243-70193115677-91112406449-61348366967-41301887681-73892593737-54039590822', '26491514771-24473569991-16656607124-07593925149-74926670900')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 253, mergeExecResult: 0. shardingNode:map[715:{SchemaIndex=179, TableIndex=715, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (533688011, 7, '17872695538-16064539790-18108982113-60646269894-74784931403-89606878485-67937667670-39824753762-37339034055-79511964331', '65227853620-62393117281-88901273019-02060144110-40123779052')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 223, mergeExecResult: 0. shardingNode:map[731:{SchemaIndex=183, TableIndex=731, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (548553435, 7, '59524856205-92318067592-84001816236-15799238765-61939312958-49774297702-51059777347-11521186806-90725070157-29567571966', '72604893873-81336794708-64989781571-89144801847-53529920629')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 296, mergeExecResult: 0. shardingNode:map[646:{SchemaIndex=162, TableIndex=646, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest4 (id, k, c, pad) VALUES (-1980507782, 8, '15361099362-33426846104-81568076007-88926275697-54147190617-21941361738-39219198670-72817831326-95768248501-29228997291', '51713925651-55077435995-56372978084-60334222441-07023013203')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 27, mergeExecResult: 0. shardingNode:map[1023:{SchemaIndex=256, TableIndex=1023, HostGroup=hostGroup4}], SQL: [INSERT INTO sbtest1 (id, k, c, pad) VALUES (419830783, 9, '39839456224-14625126106-10611694673-69035645226-20746732552-01963419766-94676550107-32542648198-96549393904-39850608367', '69725922822-08559910362-36080928509-49601235010-15261238716')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 296, mergeExecResult: 0. shardingNode:map[612:{SchemaIndex=154, TableIndex=612, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest2 (id, k, c, pad) VALUES (-1848104548, 6, '02931425025-68282480940-67294056291-41225608697-44038095895-05390291535-10414730978-25086630856-84460614365-62218384416', '74426157735-27538600335-93122572838-36921241106-44481214039')] 
WARN[0292] Slow SQL. shardingCount: 1, buildPlan: 0, getShardConns: 0, executeInMultiNodes: 305, mergeExecResult: 0. shardingNode:map[517:{SchemaIndex=130, TableIndex=517, HostGroup=hostGroup3}], SQL: [INSERT INTO sbtest5 (id, k, c, pad) VALUES (-543662597,

cobar:
sysbench --mysql-host=10.1.21.242 --mysql-port=8066 --mysql-user=zerodb --mysql-password='zerodb' --mysql-db=zerodb --threads=10 --table-size=10 --auto_inc=off --tables=5 --rand-type=uniform --db-driver=mysql --time=6000000 --events=0 --percentile=99 --report-interval=5 /usr/share/sysbench/oltp_insert.lua cleanup
sysbench --mysql-host=10.1.21.242 --mysql-port=8066 --mysql-user=zerodb --mysql-password='zerodb' --mysql-db=zerodb --threads=10 --table-size=10 --auto_inc=off --tables=5 --rand-type=uniform --db-driver=mysql --time=6000000 --events=0 --percentile=99 --report-interval=5 /usr/share/sysbench/oltp_insert.lua prepare
sysbench --mysql-host=10.1.21.242 --mysql-port=8066 --mysql-user=zerodb --mysql-password='zerodb' --mysql-db=zerodb --threads=400 --table-size=10 --auto_inc=off --tables=5 --rand-type=uniform --db-driver=mysql --time=6000000 --events=0 --percentile=99 --report-interval=5 /usr/share/sysbench/oltp_insert.lua run

zero-proxy
sysbench --mysql-host=10.1.21.242 --mysql-port=9696 --mysql-user=zerodb --mysql-password='zerodb' --mysql-db=zerodb --threads=10 --table-size=10 --auto_inc=off --tables=5 --rand-type=uniform --db-driver=mysql --time=6000000 --events=0 --percentile=99 --report-interval=5 /usr/share/sysbench/oltp_insert.lua cleanup
sysbench --mysql-host=10.1.21.242 --mysql-port=9696 --mysql-user=zerodb --mysql-password='zerodb' --mysql-db=zerodb --threads=10 --table-size=10 --auto_inc=off --tables=5 --rand-type=uniform --db-driver=mysql --time=6000000 --events=0 --percentile=99 --report-interval=5 /usr/share/sysbench/oltp_insert.lua prepare
sysbench --mysql-host=10.1.21.242 --mysql-port=9696 --mysql-user=zerodb --mysql-password='zerodb' --mysql-db=zerodb --threads=400 --table-size=10 --auto_inc=off --tables=5 --rand-type=uniform --db-driver=mysql --time=6000000 --events=0 --percentile=99 --report-interval=5 /usr/share/sysbench/oltp_insert.lua run





./benchyou --mysql-host=10.1.21.242 --mysql-port=9696 --mysql-db=zerodb --mysql-user=zerodb --mysql-password=zerodb  --oltp-tables-count=5 prepare

./benchyou --mysql-host=10.1.21.242 --mysql-port=9696 --mysql-db=zerodb --mysql-user=zerodb --mysql-password=zerodb  --oltp-tables-count=5 cleanup

./benchyou --mysql-host=10.1.21.242 --mysql-port=9696 --mysql-db=zerodb --mysql-user=zerodb --mysql-password=zerodb --ssh-user=root --ssh-password=sdfsdf --oltp-tables-count=5 --write-threads=128 --read-threads=8 --max-time=3600 random

./benchyou --mysql-host=10.1.21.242 --mysql-port=9696 --mysql-db=zerodb --mysql-user=zerodb --mysql-password=zerodb --ssh-user=root --ssh-password=sdfsdf --oltp-tables-count=5 --write-threads=128 --read-threads=8 --max-time=3600 seq


./benchyou --mysql-host=10.1.21.242 --mysql-port=9696 --mysql-db=zerodb --mysql-user=zerodb --mysql-password=zerodb --ssh-user=root --ssh-password=sdfsdf --oltp-tables-count=5 --write-threads=4 --read-threads=300 --update-threads=20 --delete-threads=10 --max-time=360000000 random


./benchyou --mysql-host=zerodb-proxy001.pre.2dfire.info --mysql-port=9696 --mysql-db=zerodb --mysql-user=zerodb --mysql-password=zerodb  --oltp-tables-count=5 prepare

./benchyou --mysql-host=192.168.0.109 --mysql-port=9696 --mysql-db=zerodb --mysql-user=zerodb --mysql-password=zerodb  --oltp-tables-count=5 prepare

./benchyou --mysql-host=zerodb-proxy001.pre.2dfire.info --mysql-port=9696 --mysql-db=zerodb --mysql-user=zerodb --mysql-password=zerodb  --oltp-tables-count=5 cleanup

./benchyou --mysql-host=zerodb-proxy001.pre.2dfire.info --mysql-port=9696 --mysql-db=zerodb --mysql-user=zerodb --mysql-password=zerodb --ssh-user=nanxing --ssh-password= --oltp-tables-count=5 --write-threads=20 --read-threads=80 --max-time=360000 random
