
---
title: About
date: 2014-03-16 10:17:16
tags: plugins
categories: Docs
description: Introduce tag plugins in freemind.
feature: images/tag-plugins/plugins.jpg
toc: true
---
# 韩志超
>男，1987.8，本科（统招），英语六级，计算机网络技术三级，6年以上测试经验，精通Python和自动化测试
>电话： 180 1018 1267   Email: Superhin@126.com    Github:  http://github.com/hanzhichao

---

## 职业技能
* 独立负责一个项目，编写测试计划/方案，推动项目及Bug解决，进行风险控制
* 熟悉OSI模型、TCP/IP、HTTP/HTTPS协议，熟悉Rest/Web Service/RPC接口
* 熟悉Linux系统，熟练使用shell编写各自脚本，熟悉Server端环境部署、监控及测试
* 熟悉分布式系统、负载均衡、高并发、高可用、微服务、事件流等基本架构
* 熟悉MySQL、Redis、ES、Mongodb，熟悉Nginx、Tomcat、ActiveMQ、阿里云MNS等中间件
* 熟悉Svn、Git/GitLab配置管理及双V、敏捷等开发流程，熟悉DevOps及Jenkins持续集成
* 2年Python使用经验，4年以上Java/C使用经验，熟悉基本数据结构和常用算法
* 测试框架搭建，熟悉Selenium2/Requests/Appium/Robot Framework，熟悉UI/接口自动化
* 熟悉Django及Rest Framework，熟悉Vue.js，Mock接口及测试工具(Web)开发
* 熟悉各种测试工具/JMeter/LR/Postman/Wrk/Fiddler/Jira/Test Link/禅道/及Zabbix/ELK/Supervisor

## 工作经历
### 2017.8至今              麻辣诱惑食品有限公司                        测试工程师
微服务测试，订单改版事件流测试，金蝶对接测试，三方外卖平台测试，微信麻小商城测试，物流外卖App测试，自动化测试框架搭建及用例编写，Jenkins自动化测试项目及环境部署

### 2016.8-2017.8           联想集团北京研究院                           测试工程师
负责ThinkStack项目自动化测试执行、Debug、批量测试结果提取，基线对比及Rally性能测试，负责虚拟环境保障及SDN网络拓扑功能测试。负责ThinkCloud大规模环境稳定性测试，大规模服务器数据层IO/CPU/Memory/MySQL压力及稳定性测试

### 2014.5-2016.7           车满满信息技术有限公司                      测试工程师
哥伦布物流管理系统（类似ERP）价格管理模块、短信定制模块，员工扣款模块测试
平安银行，虚拟银行子账户，物流助手及司机端App，第三方金主开放平台B的测试。

### 2012.3-2014.5           郑州易普格信息技术有限公司                   Web功能测试
在网站交付客户前，对客户定制版本及新功能进行包含功能、吸引性、易用性、兼容性以及前端性能进行全面测试，负责网站的交付以及客户的技术支持等工作。

## 项目经历
### 项目四：订单改版4期-事件流微服务测试   (电商)                       服务端测试                        
**项目描述：**该项目为电商项目针对日益增长的订单数据对订单表及核心业务流程的重构，对数据库订单表重新规划并采用kingshard分片，业务流程改为事件流模式

**测试环境：**CentOS 6.5 + Nginx + PHP 7.1  + 阿里云MNS + Redis + MySQL + Kingshard

**使用工具：** 阿里云MNS, Redis Desktop Manager, Supervisor, ELK, NaviCat, Tcpdump 

**职责描述：** 负责美团/饿了么/百度外卖三方平台所对接的，事件流各个微服务脚本的测试，以及综合后台物流及发票相关功能的测试，使用MNS查看每个消息队列的状态，使用Redis查看总流程状态及辅助数据，使用 Supervisor管理相关脚本，使用ELK查看日志，使用NaviCat监控数据库数据变化，必要时使用Tcpdump或MySQL通用日志辅助定位问题

### 项目二：联想集团ThinkStack/ThinkCloud项目（私有云/Laas/SDN）       服务端测试   自动化测试
**项目描述：**ThinkStack是联想研究院基于开源云计算框架定制和二次开发的一款云计算平台，主要面向Iass。注重HA高可用性，千兆网络优化，性能及大规模集群的稳定性验证。
后期和联想DCG子公司合作，切换到ThinkCloud项目，产品化，结合联想服务器及IBMX86服务器，面向
数据中心服务。注重SDN，ceph解决方案，超融合等的研究

**测试环境：**Windows 7 + Chrome + Slave:Ubuntu

**使用工具：** Xshell，Chrome，Tempest，Rally，vdbench，sysbench，iperf，stress-ng，memtester，ELK，Zabbix，Ansible + Jira + Redmine +TestLink

**职责描述：**
负责Tempest集成测试，Rally性能测试自动化的执行，调试，结果分析等，后期主要负责云计算大规模环境Data Plan(虚拟服务器层面)的稳定性测试，使用vdbench对VM Server读写进行压力测试，测试Ceph系统的稳定性，使用Sysbench，Stress-Ng，Memtester结合Zabbix监控，ELK日志收集和Ansible批量部署对虚拟机的CPU，Memory，MySQL进行压力测试

### 项目三：哥伦布物流信息管理系统（类ERP）                              系统测试
**项目描述：**哥伦布物流信息管理系统是一款SaaS，在线物流管理系统，项目前后端分离，后端采用ThinkPHP，开发RESTful接口，前端采用React，数据库采用MySQL + Elasticsearch引擎。同时项目还配套有物流助手app以及BMS后台管理系统

**测试环境：**Windows10 + Chrome + Slave: CentOS(Beta环境)

**使用工具：**Xshell，Chrome+Jsonview + 禅道

**职责描述：**
负责哥伦布价格体系的测试，使用Xshell连接Slave搭建Beta环境，使用Chrome开发者工具，抓包和定位问题。负责短信定制模块，原工作账户扣款，财务挂账生成凭证，日常收支等模块的测试工作。
参与物流助手app，金主开放平台B(满e贷)，银行账户管理方案等项目的测试与联调


## 教育/其它经历  
2005.9-2009.7               青海民族大学                                 教育技术学（教育学与计算机技术交叉学科）
2009.10-2011.12             69260部队                                      服兵役

## 部分作品
* ui_checker: 基于selenium2,page-object模式的UI自动化测试框架(Python)
* postboy: 基于json文件数据驱动的的接口测试框架(Python)
* perf_test_flat: 基于wrk + nmon + flask性能测试工具(Web)

> 详细/更多项目请参看本人GitHub:  http://github.com/hanzhichao
