---
title: MySQL Explain 整理
date: 2017-09-10 13:11:34
tags: MySQL
---

## select id

``` sql
explain select * from student where stu_id = '1000003';
explain delete from student where stu_id = '11111';
#数值越大越先执行
explain select * from (select * from student where stu_id = '1000003') tmp; 
#只有union的结果是没有 select id 的
explain select * from student where stu_id = '1000003' union select * from student where stu_id = '1000004';
```

More info: [MySQL DOC](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)