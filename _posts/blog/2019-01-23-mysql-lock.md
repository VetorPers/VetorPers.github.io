---
layout:         post
title:          mysql 事务死锁
categories: blog
description:    处理 mysql 事务死锁。
keywords: mysql,lock
---

今天在研究高并发的时候，设置了数据表的 `autocommit=0` .然后使用了 `Laravel DB::transaction()`.这样在事务结束的时候就没有提交，就产生了一个等待的事务锁锁住了表，其他对这个表做的操作都会被等待。

### 处理事务死锁

查看当前运行的所有事务
> SELECT * FROM information_schema.INNODB_TRX\G;

查看 `trx_rows_locked` 字段是否为1（表示已加锁）。
然后看事务所在的线程id `trx_mysql_thread_id`。
执行 `kill thread_id` 杀死线程。

也可以使用 `show processlist;` 查看进程情况。找到消耗资源最大的那条语句对应的id。kill 它。
