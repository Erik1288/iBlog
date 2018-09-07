---
title: MySQL-Transaction-isolation
date: 2018-09-07 10:17:32
tags:
---

### 什么是标准的事务隔离级别
事务隔离级别是来界定事务与事务之间数据可见性的程度。
在ANIS SQL标准中一共4种，按照可见性程度的从大到小，分别是：
READ UNCOMMITTED「可读未提交」
READ COMMITTED「读已提交」
REPEATABLE READ「重复一致读」
SERIALIZABLE「串行读」
不是所有的数据库都完整实现了这四种隔离级别，至于其它数据库具体的情况不在本文讨论范围内，但是MySQL中的InnoDB存储引擎很好的实现了这4种级别。
举个栗子，我们有一张balance（余额）表，

```
CREATE TABLE `mysql_learn`.`balance` (
  `id` INT NOT NULL AUTO_INCREMENT,
  `user_id` BIGINT(20) NULL,
  `count` DECIMAL(2) NULL,
  `version` BIGINT(20) NULL,
  PRIMARY KEY (`id`),
  INDEX `user_id_idx` (`user_id` ASC)
);
```

id | user_id | count | version 
----|------|---|--|
1 | 1  | 1000.00 | 1

*READ UNCOMMITTED

```
SET TX_ISOLATION = 'READ UNCOMMITTED';
```

TX A | TX B 
---|---
Begin |
select count from balance where user_id = 1; // 返回1000.00 |
| Begin
| update balance set count = 2000 where user_id = 1; |
select count from balance where user_id = 1; // 返回2000.00 |
| Rollback
select count from balance where user_id = 1; // 返回1000.00 |
Commit |

所有命令执行的顺序是从上往下的（下同），特别注意下A事务，在该事务中执行了3次「select count from balance where user_id = 1」，但每次与上一次的结果都不一样，原因是两个事务之间，隔离得「不够彻底」，当B事务还没有「Commit」或者「Rollback」的时候，A事务就能「偷窥」到它内部的数据，显然，这个数据是「脏」的（dirty），我们称这种现象为「脏读」。

*READ COMMITTED

        SET GLOBAL_TX = 'READ COMMITTED';

TX A | TX B 
---|---
Begin |
select count from balance where user_id = 1; // 返回1000.00 |
| Begin
| update balance set count = 2000 where user_id = 1;
select count from balance where user_id = 1; // 返回1000.00 |
| Commit
select count from balance where user_id = 1; // 返回2000.00 |
加一个update语句 |
Commit |

这一次，我们再次关注A事务，在B事务没有提交的时候，A事务读取到的值始终是1000，而当B事务提交了数据更新时，即使A事务还没有提交，也可以「Read」到B事务「Commited」的数据变化，符合该级别的名称，这种现象也叫幻读「Phantom Problem」，专业的解释是指在同一事务下，连续执行两次同样的SQL语句可能会导致不同的结果。所以这种隔离级别可以去实现乐观锁。但是，熟悉MySQL的人都知道它的默认级别「REPEATABLE READ」，与上面的例子一样的情况下，会有不一样的表现，下个例子说明。

TX A | TX B 
---|---
Begin |
select count from balance where user_id = 1; // 返回1000.00 |
update balance set count = 2000 where user_id = 1; |
| Begin
| update balance set count = 3000 where user_id = 1;
select count from balance where user_id = 1; // 返回2000.00 |
| Commit
select count from balance where user_id = 1; // 返回3000.00 |
Commit |

*REPEATABLE READ

        SET @@TX_ISOLATION = 'REPEATABLE READ';

TX A | TX B | TX C
---|---|---
Begin || 
select count from balance where user_id = 1; // 返回1000.00 ||
| Begin |
| update balance set count = 2000 where user_id = 1; |
select count from balance where user_id = 1; // 返回1000.00 ||
| Commit
||Begin
|| select count from balance where user_id = 1; // 返回2000.00
select count from balance where user_id = 1; // 返回1000.00 ||
Commit ||Commit

当要说明这个隔离级别的时候，我们又加了一个C事务，先从A和B事务入手，与「READ COMMITTED」唯一的区别是，在B事务提交了「update balance set count = 2000 where user_id = 1」之后，A事务提交之前，A事务「Repetitive」得「Read」，得到的结果却总是一致的，同样在这个时候，C事务进来，但C事务却能读到B事务已经提交的数据更新。总结下，该级别的特点就是在同一个事务中，同一条SELECT语句读到的数据总是一致的，即使其它的事务已经对SELECT的记录修改并且提交了，这样就解决了幻读的问题。
*问题来了，为什么设计者需要把默认隔离级别设置成「REPEATABLE READ」，事务A和B中对于同一条记录有两个不同版本，又是怎么实现的，且听后文分解「TBD」。

TX A | TX B 
---|---
Begin |
select count from balance where user_id = 1; // 返回1000.00 |
update balance set count = 2000 where user_id = 1; |
| Begin
| update balance set count = 3000 where user_id = 1; //阻塞
select count from balance where user_id = 1; // 返回2000.00 |
select count from balance where user_id = 1; // 返回3000.00 |
Commit |
| Commit

*SERIALIZABLE
由于是串行的，所以高并发下性能有很大问题。