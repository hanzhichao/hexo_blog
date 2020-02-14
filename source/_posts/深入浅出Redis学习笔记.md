---
layout: post
title: 深入浅出Redis学习笔记
comments: true
toc: true
date: 2018-02-01 20:03:11
updated: 2018-02-01 20:03:11
categories:
tags:
---

    1. Redis的发展史

        1. Redis[Remote Directory Server]:远程服务字典
    2. 下载安装Redis

        1. Linux下安装Redis
http://redis.io/dowload
wget http://download.redis.io/releases/redis-3.0.0.tar.gz
tar xzf redis-3.0.0.tar.gz
cd redis-3.0.0
make
        2. 在bin可执行的程序

            * redis-server: Redis服务器
            * redis-cli:命令行客户端
            * redis-benchmark:Redis的性能测试工具
            * redis-check-aof:AOF文件修复工具
            * redis-check-dump: RDB文件检测工具


            * redis.conf 时Redis的配置文件
将配置文件中daemonize yes  以守护进程的方式来使用
        3. 启动

            1. 直接启动

                * redis-server
                * 指定配置文件 redis-server /etc/redis/redis.conf
                * 指定端口  redis-server  /etc/redis/redis.conf --port 6370(会覆盖redis.conf中port的设置）
客户端连接 redis-cli -h localhost -p 6370
            2. 停止redis

                1. 客户端输入 shutdown关闭redis-server
                2. 结束redis的进程也可以
        4. 

    3. 命令返回值

        1. 状态回复

            1. ping          -PONG
            2. SET test 'this is a test'     -OK
            3. 和SQL命令一样，Redis命令不区分大小写，建议系统命令使用大写，和名字有关的使用小写
        2. 错误回复

            1. 错误回复以（error)开头
(error) ERR unknown command 'PONG'
        3. 整数回复

            1. 以interger 数值
DBSIZE       --（integer) 2
        4. 字符串回复

            1. GET  test 
            2. (nil)代表空的结果
        5. 多行字符串回复

            1. KEYS *  得到当前数据库中存在的键名
    4. Redis配置选项相关内容

        1. 动态设置/获取配置选项的值

            1. 获取 CONFIG GET name
CONFIG GET port
1) "port"
2) "6379"
            2. 设置 CONFIG SET name value
CONFIT SET loglevel warning
OK
        2. Redis 配置文件redis.conf选项相关

            * --链接选项
            * port 6379 默认端口
            * bind 127.0.0.1 默认绑定的主机地址
            * timeout 0, 当客户端闲置多就之后关闭链接，0代表美哟启动这个选项
            * loglevel notice, 日志的级别

                * debug: 很详细的信息，适合开发和测试
                * verbose： 包含很多不太有用的信息
                * notice： 比较适合生产环境
                * warning： 警告信息
            * logfile stdout,日志的记录方式， 默认为标准输出
            * datebases 16, 默认数据库的数量，默认数据库的编号从0开始
            * save
            * -- 快照
            * save <seconds> <changes>: 多少秒有多少次改变将其同步到磁盘中的数据文件中
save 900 1 ---900s 1个更改
save 300 10
save 60   10000
            * dbfilename   dump.rdb, 指定本地数据库文件名，默认为 dump.rdb
            * dir ./ ,指定本地数据库的存放目录，默认时当前目录
    5. Redis数据类型

        1. String

            1. SET key [EX seconds] [PX mileseconds] [NX|EX]
            2. GET key
            3. MSET
            4. MGET
            5. DEL
        2. Integer

            1. INCR
            2. INCRBY  5
            3. INCRBYFLOAT 1.2
            4. DECR
            5. DECRBY
        3. Hash

            1. HSET key field
            2. HGET key field
            3. HMSET
            4. HMGET
            5. HGETALL
            6. HKEYS
            7. HDEL
        4. List
        5. Set
        6. Sorted Set
    6. KEYS相关命令

        1. KEYS *

            1. * 任意多个字符
            2. ？ 任意一个
            3. []--范围内任一个 a[b-z]
            4. \ --匹配特殊字符
        2. EXISTS
        3. EXPIRE
        4. EXPIRENX
        5. PEXPIRE
        6. TTL
        7. PTTL
        8. PERMANT

