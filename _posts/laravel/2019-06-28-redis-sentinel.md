---
layout:         post
title:          redis 哨兵模式及在 laravel 中的使用
description:    在laravel中使用 redis 的哨兵.
categories: laravel
keywords: laravel ,redis
--- 

### 主从配置(master-slave)

* 复制 redis 配置文件以开启多个 slave

> sudo cp /etc/redis.conf /etc/redis-6381.conf

> sudo cp /etc/redis.conf /etc/redis-6382.conf

* 编辑 slave 配置文件，主要修改参数

```

port 6381

pidfile "/var/run/redis-6381.pid"

logfile "/var/log/redis/redis-6381.log"

slaveof 11.11.11.11 6381

masterauth "123456" # 主从都保持一样的密码，且 master 的配置也需要这一行，在执行切换 master 的时候好像不会去添加这一行

```

* /usr/bin/redis-server /etc/redis.conf 通过配置启动 redis

### 哨兵配置(sentinel)

* 复制哨兵配置，这儿开启3个哨兵

> sudo cp /etc/redis-sentinel.conf /etc/redis-sentinel-26381.conf

> sudo cp /etc/redis-sentinel.conf /etc/redis-sentinel-26382.conf

* 编辑哨兵配置文件，主要修改参数如下，根据具体情况配置

```

port 26381

pidfile "/var/run/redis-sentinel-26381.pid"

logfile "/var/log/redis/redis-sentinel-26381.log"

sentinel monitor mymaster 11.11.11.11 6379 2 #主节点别名为mymaster，后面是ip和端口，2代表判断主节点失败至少需要2个sentinel节点同意

sentinel auth-pass mymaster 123456

sentinel down-after-milliseconds mymaster 30000 #主节点故障30秒后启用新的主节点

sentinel parallel-syncs mymaster 1 #故障转移时最多可以有1个从节点同时对主节点进行数据同步，数字越大，用时越短，存在网络和 IO 开销

sentinel failover-timeout mymaster 180000 #故障转移超时时间180s：a 如果转移超时失败，下次转移时时间为之前的2倍；b 从节点变主节点时，从节点执行 slaveof no one 命令一直失败的话，当时间超过180S时，则故障转移失败；c 从节点复制新主节点时间超过180S转移失败

```

* /usr/bin/redis-sentinel /etc/redis-sentinel.conf 通过配置启动哨兵

### laravel 哨兵配置

```

'default' => [
            'tcp://11.11.11.11:26379',
            'tcp://11.11.11.11:26381',
            'tcp://11.11.11.11:26382',    //这3个都是sentinel节点的地址
            'options' => [
                'replication' => 'sentinel',
                'service'     => env('REDIS_SENTINEL_SERVICE', 'mymaster'),    //sentinel
                'parameters'  => [
                    'host'     => env('REDIS_HOST', '127.0.0.1'),
                    'port'     => env('REDIS_PORT', 6379),
                    'password' => env('REDIS_PASSWORD', null),    //redis的密码,没有时写null
                    'database' => 0,
                ],
            ],
        ]

```
