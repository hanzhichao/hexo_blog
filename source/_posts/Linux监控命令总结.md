---
layout: post
title: Linux监控命令总结
comments: true
toc: true
date: 2018-02-01 20:04:10
updated: 2018-02-01 20:04:10
categories:
tags: Linux, 性能测试
---

Linux命令

Linux命令

1 Linux基本命令
  1.1 目录操作
    1.1.1 ls
    1.1.2 cp
        1.1.2.1 -R
    1.1.3 mv
    1.1.4 chmod
  1.2 文件操作
    1.2.1 touch
    1.2.2 vim
    1.2.3 cat/tac/more/less
    1.2.4 head
        1.2.4.1 head -2 file 查看前两行
        1.2.4.2 head -c 2048 file 查看前2048kb
    1.2.5 tail
        1.2.5.1 tail -100 file 查看末尾100行
        1.2.5.2 tail -f /tmp/ERROR.log 事实查看文件更新
    1.2.6 清空文件
        1.2.6.1 cat /etc/null > file
        1.2.6.2 echo /etc/null >file
        1.2.6.3 echo "" >file
    1.2.7 find
        1.2.7.1 -name 指定文件名，支持*
        1.2.7.2 -user 制定所属用户
        1.2.7.3 find / -name cat*.h -user tomcat
  1.3 重启/关机
    1.3.1 shutdown
        1.3.1.1 -h
        1.3.1.2 -r
    1.3.2 reboot
  1.4 解压/安装
    1.4.1 tar
        1.4.1.1 -x 解压 tar
                1.4.1.1.1 tar -xvf file.tar
        1.4.1.2 -v 解压可视
        1.4.1.3 -f 文件
                1.4.1.3.1 必须
        1.4.1.4 -z 解压tar.gz
                1.4.1.4.1 tar -zxvf file.tar.gz
        1.4.1.5 -j 解压 tar.bz2
                1.4.1.5.1 tar -jxvf file.tar.bz2
        1.4.1.6 -Z 解压 tar.Z
                1.4.1.6.1 tar -Zxvf file.tar.Z
        1.4.1.7 其他
                1.4.1.7.1 -c建立压缩
                1.4.1.7.2 -t 查看内容
                1.4.1.7.3 -r 向压缩包追加文件
                1.4.1.7.4 -u 跟体系元压缩包中的文件
                1.4.1.7.5 - O 将文件纪要的标准输出
    1.4.2 unrar
        1.4.2.1 unrar e file.rar
    1.4.3 unzip
        1.4.3.1 unzip file.zip
    1.4.4 rpm
        1.4.4.1 -ivh
        1.4.4.2 -a
        1.4.4.3 -q
        1.4.4.4 -l
        1.4.4.5 rpm -qal | grep mysql
        1.4.4.6 rpm -qa | grep yum | xargs rpm -e --nodeps
  1.5 查看系统版本
    1.5.1 uname -a
    1.5.2 lbs_release -a
    1.5.3 cat /proc/version
2 Linux监控命令
  2.1 综合
    2.1.1 top
        2.1.1.1 实时监控系统状态，并可以按照cpu/mem/执行时间排序
        2.1.1.2 启动参数
                2.1.1.2.1 -b 批次运行
                2.1.1.2.2 -d 延迟时间
                2.1.1.2.3 -n 运行次数
                2.1.1.2.4 -u/U 监控制定用户
                2.1.1.2.5 -p 监控制定进程
                2.1.1.2.6 -H 显示线程
                2.1.1.2.7 -i 显示空闲的进程
        2.1.1.3 任务区命令
                2.1.1.3.1 任务排序
                2.1.1.3.1.1 C 按cpu使用率排序，再按一下反向排序
                2.1.1.3.1.1.1
                2.1.1.3.1.2 M 按内存使用排序
                2.1.1.3.1.3 T 按运行时间排序
                2.1.1.3.2 s
                2.1.1.3.2.1 更改delay
                2.1.1.3.3 1
                2.1.1.3.3.1 显示所有cpu
        2.1.1.4 VIRT
                2.1.1.4.1 程序向系统申请的内存
        2.1.1.5 RES
                2.1.1.5.1 程序实际使用内存
        2.1.1.6 Swap
                2.1.1.6.1 程序实际使用的虚拟内存
                2.1.1.6.2 添加Swap列
                2.1.1.6.2.1 按f,选中Swap按空格(先删除1列）按s,按q
    2.1.2 vmstat
        2.1.2.1 监控操作系统的进程状态、内存、虚拟内存、磁盘IO、上下文、CPU的信息
    2.1.3 sar
        2.1.3.1 全名的获取到cpu 、运行、磁盘IO、虚拟内存、内存、网络等信息
        2.1.3.2 sar -u 1 3
        2.1.3.3 -u cpu
        2.1.3.4 -v 进程 节点 文件 锁表
        2.1.3.5 -d 硬盘使用
        2.1.3.6 -r 内存
        2.1.3.7 -n 网络使用
        2.1.3.8 -R 进程活动
    2.1.4 dstat
        2.1.4.1 监控系统cpu,网络,磁盘,页面交换
                2.1.4.1.1 yum install dstat
                2.1.4.1.2 可以展示最好各项资源的进程，可以生成csv文件
        2.1.4.2 参数
                2.1.4.2.1 -c 总cpu状态
                2.1.4.2.1.1 dstat -clp 1 10
                2.1.4.2.1.2 -l load
                2.1.4.2.1.3 -p process status
                2.1.4.2.1.4 -C 0,1 监控指定cpu(0,1)的状态
                2.1.4.2.2 -m --mem 内存
                2.1.4.2.3 -d 磁盘
                2.1.4.2.3.1 dstat -d --disk-util --disk-tpk 2 10
                监控CPU进行I/O读写：cpu监控里面的wai增大，势必会造成，进程I/O读写频繁。Wait次数越多，某个进程的I/O就会越大。某个进程的I/O越大，I/O读写的tps就越频繁。（CPU监控）wait block i/o process 和（磁盘监控）read writ|util|reads writs指标，成正比。
                2.1.4.2.3.2 --disk-util 磁盘使用率
                2.1.4.2.3.3 --disk-tps 每秒tps read write 次数
                2.1.4.2.4 -n --net 网络
                2.1.4.2.5 --top-cpu
                2.1.4.2.5.1 列出最耗CPU的进程
                2.1.4.2.6 --top-bio
                2.1.4.2.6.1 列出当前CPU进程里面，I/O频繁，并且是受阻的
                2.1.4.2.7 --top-mem
                2.1.4.2.7.1 展示最耗内存的进程，耗用了多少内存
                2.1.4.2.8 --output file 写csv文件
        2.1.4.3 案例
                2.1.4.3.1 dstat 2 3
                2.1.4.3.1.1 默认 -cdngy
                2.1.4.3.2 dstat -almpsr --disk-utl --top_bo --top-cpu --top-men --output /root/dstat.csv 3 3600&
                2.1.4.3.3 dstat -clpmdn --top-cpu --top-bio --top-men --disk-util --disk-tps --output /root/dstat2.csv 3 1200
  2.2 cpu
    2.2.1 uptime
        2.2.1.1 统计系统当前的运行状态
                2.2.1.1.1 查看当前CPU的load，同dstat -l
    2.2.2 tload
        2.2.2.1 查看当前CPU的load，每隔2到3s更新一次
    2.2.3 cat /proc/loadavg
    2.2.4 mpstat
        2.2.4.1 输出每个CPU的运行状况，为多处理器系统中的CPU利用率提供统计信息
  2.3 mem
    2.3.1 free
        2.3.1.1 监控系统内存
        2.3.1.2 -b/k/m/g/--tera 设定单位为b/Kb/M/G/T
        2.3.1.3 -l 显示 low/high
        2.3.1.4 -t 显示total
        2.3.1.5 -s 更新时间间隔
        2.3.1.6 -c 更新次数
        2.3.1.7 free -m -t -s 1 -c 3
        2.3.1.8 实际已使用物理内存=used(Mem)-buff/cache
        2.3.1.9 实际空闲物理内存=free(Mem)+buff/cache
        2.3.1.10 如果虚拟内存，使用率持续增大，有可能，真实使用的物理内存越来越小，可能不够。
  2.4 process
    2.4.1 ps
  2.5 disk/io
    2.5.1 df
        2.5.1.1 文件系统磁盘空间的使用报告
        2.5.1.2 -a 显示所有文件系统
    2.5.2 iostat
        2.5.2.1 磁盘监控
                2.5.2.1.1 yum -q install /usr/bin/iostat
        2.5.2.2 参数
                2.5.2.2.1 -c 只显示cpu
                2.5.2.2.2 -d 只显示io信息
                2.5.2.2.3 -t 显示时间戳
                2.5.2.2.4 -x 把/proc/diskstats的内容显示出来
                2.5.2.2.5 -k 以kb单位显示
                2.5.2.2.6 iostat -k -d -x 3 1
        2.5.2.3 字段解析
                2.5.2.3.1 rrqm/s
                2.5.2.3.1.1 每秒进行 merge 的读操作数目
                2.5.2.3.2 wrqm/s
                2.5.2.3.2.1 每秒进行 merge 的写操作数目
                2.5.2.3.3 r/s
                2.5.2.3.3.1 每秒完成的读 I/O 设备次数
                2.5.2.3.4 w/s
                2.5.2.3.4.1 每秒完成的写 I/O 设备次数
                2.5.2.3.5 rkB/s
                2.5.2.3.5.1 每秒读K字节数
                2.5.2.3.6 wkB/s
                2.5.2.3.6.1 每秒写K字节数
                2.5.2.3.7 avgrq-sz
                2.5.2.3.7.1 平均每次设备I/O操作的数据大小
                2.5.2.3.8 avgqu-sz
                2.5.2.3.8.1 平均I/O队列长度
                2.5.2.3.9 await
                2.5.2.3.9.1 平均每次设备I/O操作的等待时间 (毫秒)
                2.5.2.3.10 svctm
                2.5.2.3.10.1 平均每次设备I/O操作的服务时间 (毫秒)
                2.5.2.3.11 %util
                2.5.2.3.11.1 一秒中有百分之多少的时间用于 I/O 操作
        2.5.2.4 分析
        只要读写次数上升（r/s和w/s；rKB/s和wKB/s），会引起读写I/O的平均排队数上升（avgqu-sz），引起I/O等待（await）上升，引起磁盘读写的时候磁盘服务时间增大（svctm），当这些都增大，会引起磁盘使用率增大（%util）。如何让使用率下降，程序I/O要降低，要不然磁盘读写频繁是不会下降的。项目经验：开发的代码没问题，但频繁的往磁盘写日志，并发量越大，写磁盘的日志就越频繁，造成整体性能下降。
    2.5.3 iotop
        2.5.3.1 基础的I/O监控命令
                2.5.3.1.1 yum install iotop
        2.5.3.2 -d 采集频率
  2.6 network
    2.6.1 ifconfig
        2.6.1.1 分网卡看进出包的个数、字节数
                2.6.1.1.1 yum install net-tools
        2.6.1.2 RX packets 接收数据包
        2.6.1.3 TX packets 发送数据包
    2.6.2 ifstat
        2.6.2.1 统计网络接口活动状态
    2.6.3 netstat
        2.6.3.1 显示本机网络链接、运行端口、路由表等信息
十一月 21, 2017. Created by XMind