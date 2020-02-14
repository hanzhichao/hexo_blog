---
layout: post
title: MySQL高性能优化
comments: true
toc: true
date: 2018-02-01 19:59:07
updated: 2018-02-01 19:59:07
categories:
tags: 性能优化, MySQL
---

    1. 回答要具体
    2. 

    3. 建表原则

        1. 定长与变长分离
        2. 常用字段合不常用字段分离
        3. 在一对多，需要统计的字段上添加冗余
    4. 列类型选择

        1. 字段类型优先级

            1. 整形>date,time>enum,char>varchar>blob,text
            2. time定长，为了方便查询可以用时间戳
            3. enum定长，需要转为整形
            4. char定长，需要考虑字符集和校对集（排序）
            5. varchar，不定长
            6. blob,text 不定长
            7. 例如性别sex字段

                1. tinyint unsigned not null  定长1字节
                2. char（1） 需要考虑字符集和校对集
                3. emum('男','女'）多了一个转换过程
        2. 够用就行（如smallint,varchar(N)）
        3. 尽量避免用Null(不方便查询，需要is Null,而不是=）
    5. btree索引（类似二分查找法）

        1. 42亿数据，顺序查找法平均查找21亿次，btree查找32次（2^32-1>42亿）
    6. hash索引

        1. 优点：地址是算出来的，只需要一次查找
        2. 缺点：地址不连续，不适用范围查找id>3，模糊查找name like "a%"
    7. 建立索引常见误区
    8. 索引经典题目
    9. 索引试验
    10. 聚簇索引与非聚簇索引

        1. innodb--聚簇索引，数据在索引叶子底下，不用回行，读取磁盘，插入时插入到索引指定位置，非主键索引，返回对主键的引用
        2. myisam--非聚簇索引，索引与数据分离，返回数据地址，插入时无序
    11. 页分裂试验
    12. 索引覆盖

        1. 联合索引中有需要查找的值，不用回行
    13. 某论坛经典题目
    14. 理想的索引

        1. 识别度高
        2. 字段短
    15. 伪哈希索引
    16. 多列索引原则
    17. 索引与排序
    18. 重复索引与冗余索引
    19. 修复表
    20. 查询大原则
    21. explain详解

        1. explain ....\G
    22. in型查询陷阱

        1. select goods_id,cat_id,goods_name from goods where cat_id in (select cat_id from category where parent_id=6)
        2. 误区：给我的感觉是，先查到内城的6号栏目的子栏目，然后外层cat_id in (7,8,9)
        3. 事实，外层每找到一个到内层对比
    23. 强制使用索引
    24. count和union
    25. limit翻页优化专题

        1. limit offset N,当offset非常大时，效率极低，原因是mysql并不是跳过offset行，然后单取N行，而是取offset+N行，返回放弃前offset行，返回N行，效率较低，当offset越大时，效率越低
        2. 优化办法
        3. 从业务上去解决，不允许翻过超过100页

            1. 2.不用offset,用条件查询 select id,nmae from lx_come limit 500000,10->select id,name from lx_come where id>500000 limit 10
    26. 强制使用索引

        1. select id from it_area use index(primary) where pid=60 limit 1

