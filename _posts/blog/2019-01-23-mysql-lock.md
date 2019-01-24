---
layout:         post
title:          mysql 事务死锁，开启慢查询日志
categories: blog
description:    处理 mysql 事务死锁，开启慢查询日志。
keywords: mysql,lock，slow
---

今天在研究高并发的时候，设置了数据表的 `autocommit=0` .然后使用了 `Laravel DB::transaction()`.这样在事务结束的时候就没有提交，就产生了一个等待的事务锁锁住了表，其他对这个表做的操作都会被等待。

### 处理事务死锁

查看当前运行的所有事务
> SELECT * FROM information_schema.INNODB_TRX\G;

查看 `trx_rows_locked` 字段是否为1（表示已加锁）。
然后看事务所在的线程id `trx_mysql_thread_id`。
执行 `kill thread_id` 杀死线程。

也可以使用 `show processlist;` 查看进程情况。找到消耗资源最大的那条语句对应的id。kill 它。


### 开启慢查询

#### 简介
开启慢查询日志，可以让MySQL记录下查询超过指定时间的语句，通过定位分析性能的瓶颈，才能更好的优化数据库系统的性能。

#### 参数说明
slow_query_log 慢查询开启状态
slow_query_log_file 慢查询日志存放的位置（这个目录需要MySQL的运行帐号的可写权限，一般设置为MySQL的数据存放目录）
long_query_time 查询超过多少秒才记录

#### 设置步骤
1.查看慢查询相关参数

> mysql>show variables like 'slow_query%';

> mysql>show variables like 'long_query_time';

2.设置方法

方法一：全局变量设置
将 slow_query_log 全局变量设置为“ON”状态

> mysql> set global slow_query_log='ON'; 
设置慢查询日志存放的位置

> mysql> set global slow_query_log_file='/usr/local/mysql/var/slow.log';
查询超过1秒就记录

> mysql> set global long_query_time=1;

方法二：配置文件设置
修改配置文件my.cnf，在[mysqld]下的下方加入

 ```
[mysqld]
slow_query_log = ON
slow_query_log_file = /usr/local/mysql/data/slow.log
long_query_time = 1
```

3.重启MySQL服务

service mysqld restart

#### 测试
1.执行一条慢查询SQL语句

> mysql> select sleep(2);
2.查看是否生成慢查询日志

`ls /usr/local/mysql/data/slow.log`
如果日志存在，MySQL开启慢查询设置成功！
