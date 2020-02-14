---
title: tomcat
date: 2017-11-02 20:50:35
categories: 性能调优
tags: tomcat
toc: true
---
# Tomcat 调优总结
>tomcat 优化分为系统优化、JVM调优、tomcat本身优化

## 系统优化
### 调整tomcat内存占用
```
  vim /usr/share/tomcat8/bin/catalina.sh
  设置JAVA_OPT="-Xms 1024m -Xmx 1520m"
  -Xms tomcat启动初始内存，一般为服务器开机空闲内存-100M
  -Xmx tomcat最大占用内存
```
## JVM优化
```
  JVM堆的大小决定了垃圾回收的时间和频度。
  堆设置的很大，完全垃圾收集就会很慢，频度会降低，相反，完全收集就很快，但是会更频繁。
  调整目标：最小化垃圾收集时间，一在特定的时间内最大化处理客户请求
  一般堆设置为物理内存的80%,一次完全的垃圾收集不应超过3-5s
  当增加处理器时，需要增加内存，应为内存分配时并行的，而垃圾回收不是并行的

```
## tomcat本身优化
```
  vim /etc/tomcat8/server.xml
  <Connector />配置中
  maxThreads="150" 最大线程数，window最大是2000，linux最大是1024
  minSpareThreads="25" 最小空等待线程-没有连接时也开启25个空线程等待
  maxSpareThreads="75" 最多空线程数，一旦超过这个值，tomcat就会关闭不再需要的socket线程
  acceptCount="100" 最大排队数量，当同时连接数量达到maxThreads时，还可以接收排队的连接数量，超过这个数量，直接返回拒绝连接
  enableLookups=false 是否反查域名，配置为false可以提高处理能力
  connectionTimeout=30000 网络连接超时，单位：毫秒，设置为0表示永不超时
  maxKeepAliveRequest="1" 等待响应次数，每个连接只响应一次就关闭，这样就不会等待timeout,而产生大量的TIME_WAIT连接
  buffersize="2048"  输入流缓冲大小
  compression="off"  压缩传输，取值on/off/force
```
最新的tomcat7,tomcat8只剩下两个参数
