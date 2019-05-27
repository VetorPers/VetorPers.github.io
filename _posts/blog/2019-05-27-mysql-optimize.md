---
layout:         post
title:          记一次我的 mysql 调优经历
categories: blog
description:    mysql 优化
keywords: mysql 优化 索引 join 排序 explain
---

某一天突然发现我们的管理后台的一个请求很慢，接口调用时间达到了8 s。很纳闷，一个简单的用户列表接口，用户数据才 4k+，还使用了分页，为什么会这么慢呢。

经过调试发现是 mysql 执行时间太长。这儿我们模拟两张表和表数据：
```
create table `users` (
  `id` int(10) unsigned not null auto_increment,
  `name` varchar(255) default 'name',
  `a` varchar(255) default 'aaaaaaaaaaaaaaaa',
  `b` varchar(255) default 'bbbbbbbbbbbbbbbb',
  `c` varchar(255) default 'cccccccccccccccc',
  primary key (`id`)
) engine=innodb;
delimiter ;;
  create procedure usersdata()
  begin
    declare i int;
    set i=1;
    while(i<=4000)do
      insert into users(`name`) values('name');
      set i=i+1;
    end while;
  end;;
delimiter ;
call usersdata();
create table `user_enterprises` (
  `id` int(11) unsigned not null auto_increment,
  `user_id` int(11) default null,
  primary key (`id`)
) engine=innodb;
delimiter ;;
  create procedure enterprisesdata()
  begin
    declare i int;
    set i=1;
    while(i<=4000)do
      insert into user_enterprises(`user_id`) values(i);
      set i=i+1;
    end while;
  end;;
delimiter ;
call enterprisesdata();

```

把我们需要执行的 sql 打印出来：
```
select * from `users` left join `user_enterprises` on `users`.`id` = `user_enterprises`.`user_id` order by `users`.`id` desc;
```

使用 explain 命令：￼￼
![记一次我的 MySQL 调优经历](https://iocaffcdn.phphub.org/uploads/images/201905/27/6618/pG8eE9YXNI.png!large)

我们看到在 `join` 的时候使用了 `BNL` 算法。它的过程大概是：
* 首先把 `users` 表的所有数据加入到 join buffer 中。
* 扫描整个 `user_enterprises` 表的每一行数据，与 join buffer 中的 `users` 数据作对比，将满足条件的加入结果集。

虽然操作量很大，但都是在内存中完成的。
查询扫描行数：
```
/* 打开 optimizer_trace，只对本线程有效 */
SET optimizer_trace='enabled=on'; 

/* @a 保存 Innodb_rows_read 的初始值 */
select VARIABLE_VALUE into @a from  performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 执行语句 */
select * from `users` left join `user_enterprises` on `users`.`id` = `user_enterprises`.`user_id` order by `users`.`id` desc;

/* @b 保存 Innodb_rows_read 的当前值 */
select VARIABLE_VALUE into @b from performance_schema.session_status where variable_name = 'Innodb_rows_read';

/* 计算 Innodb_rows_read 差值 */
select @b-@a;
```
我们得到的扫描行数是 12000。在执行 join 的时候，扫描行数应该是 4000 + 4000，还有 4000 的扫描行数我推测应该是回表取了数据。

启用 `optimizer_trace` 调试：
```
/* 打开 optimizer_trace，只对本线程有效 */
set optimizer_trace='enabled=on'; 

/* 执行语句 */
select * from `users` left join `user_enterprises` on `users`.`id` = `user_enterprises`.`user_id` order by `users`.`id` desc;

/* 查看 OPTIMIZER_TRACE 输出 */
select * from `information_schema`.`optimizer_trace`;
```
我们看到使用的排序方法是 `rowid` 排序，`select @@max_length_for_sort_data`  的结果为 1024，即参与排序的字段大于了这个值，mysql 会把排序字段和主键取出来放入 sort buffer，完成排序后回表取数据。在这儿还用到了临时表。所以大致执行过程应该是 join 之后把数据存在了临时表，然后使用 `rowid` 排序。

从上面我们发现这个过程是复杂的，如果在 `user_enterprises` 表上给 `user_id` 加上索引。
```
alter table `user_enterprises` add key `user_id_index` (`user_id`);
```

再次使用 explain 查看结果：

![记一次我的 MySQL 调优经历](https://iocaffcdn.phphub.org/uploads/images/201905/27/6618/8RcAfyMgVI.png!large)

首先 join 的执行流程发生了变化，大体流程是：
* 在 users 表里取出一行数据
* 根据索引在 user_enterprises 表里获取结果，组成结果集

我们发现使用到了索引后，就没有在 join buffer 里那些复杂操作了。因为索引的有序性，排序也免了，整个查询过程所需要的时间也大大减少。

~由此可见索引是多么的重要啊！！！
