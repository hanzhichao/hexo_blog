---
layout: post
title: Bootstrap3入门学习笔记
comments: true
toc: true
date: 2018-02-01 19:58:40
updated: 2018-02-01 19:58:40
categories:
tags: 前端
---

    1. 安装与引用

<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="stylesheet" href="css/bootstrap.min.css"/>
<script type="application/javascript" src="js/jquery-1.11.1.min.js"></script>
<script language="Javascript" src="js/bootstrap-min.js"></script>


    1. 栅格系统布局

        1. 栅格系统简介

            1. 响应式设计
            2. 栅格实现原理
            3. 媒体查询
        2. 栅格布局基本用法

            1. 布局容器container
            2. 列组合 .col-md-*
            3. 列偏移 .col-md-offset-*
            4. 列嵌套
            5. 列排序 .col-md-push-* .col-md-push-*
        3. 栅格参数

            1. 跨设备组合定义
            2. 清除浮动 clearfix visible-xs
        4. 禁止响应布局

            1. 删除view的meta
            2. 为container设置固定的宽度
            3. 对导航移除折叠和展开行为
    2. 文本排版

        1. 排版前的基础

            1. html5文档类型
            2. 移动设备优先
            3. 响应式图片
            4. 排版与链接
        2. 标题 h1-h6 small
        3. 页面主题 body 全局样式 p全局样式 对齐方式
        4. 强调文本 small strong em cite
        5. 缩略语abbr
        6. 地址元素address
    3. 列表与代码

        1. 列表 

            1. 无序列表
            2. 有序列表
            3. 去点列表 .list-unstyled
            4. 内联列表 .list-inline
            5. dl列表
            6. 水平列表dl.dl-horizontal
        2. 代码

            1. <code>单行内联代码
            2. <kbd>输入键
            3. <pre>多行代码块
            4. <samp>程序结果
    4. 表格样式

        1. 基础样式
        2. 带条纹.table-striped
        3. 带边框.table-border
        4. 悬停.table-hover
        5. 紧凑 .table-condensed
        6. 行样式.active .success .info
        7. 响应式表格
    5. 表单样式

        1. 基础样式

            1. 全局样式
            2. .form-control
            3. .form-group
        2. 内联表单 form-inline
        3. 横向表单 .form-horizontal
        4. 表单操作

            1. input
            2. select
            3. textarea
            4. checkbox&radio
            5. 静态控件
        5. 控件类型

            1. 焦点状态
            2. 禁用状态
            3. 被警用的fieldset
            4. 只读状态
            5. 校验状态
        6. 添加额外的图标
        7. 控件大小
        8. 辅助文本
    6. 按钮样式
    7. 图片样式

