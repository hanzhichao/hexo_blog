---
layout: post
title: jQuery入门学习笔记
comments: true
toc: true
date: 2018-02-01 19:57:58
updated: 2018-02-01 19:57:58
categories:
tags: 前端
---
    1. 安装入门
    2. 选择器初步

        1. Basics 基本选择器

            * $('code')  选择基本标签
            * $('#myid') 选择id=myid的内容
            * $(.myclass) 选择class=myclass的内容
            * $(*) 选择所有
            * $('code, #myid, .myclass) 选择 code id=myid 及class=myclass的所有
        2. Hierarchy 层级选择器

            * $('div code') 选择div 下的 code
            * $('li > ul')选择 li 下的直接 子元素 ul  子元素选择器
            * $('strong + em')选择 strong之后第一个em  姊妹选择器
            * $('strong ~ em') 选择strong 之后的所有 em 姊妹选择器
        3. Basic Filters 基本过滤器

            * $('li:first')  第一个li
            * $('li:last') 最后一个
            * $('li not(li:first)') 不是
            * $('li:even') 偶数，从0开始
            * $('li:odd') 奇数
            * $('li:eq(1)') 等于
            * $('li:gt(2)') 大于
            * $('li:lt(2)') 小于
            * $(':header') ??
            * $(':animated') ???
    3. 选择器高阶

        1. Content Filters 内容选择器

            * $('li:contents(aaa)') li中包含字符 aaa
            * $(':empty')    <></>之间内容为空的
            * $('li:has(a)')  下面包含 a
            * $('p:parent') 父级选择器
        2. Visbility Filters 可见选择器

            * $(':hidden') 选择隐藏的
            * $(':visible') 选择可见的
        3. Attribute Filters 属性选择器

            * $('li[class]')
            * $('a[ref="self"]')
            * $('a[ref!="nofollow"]')
            * $('[class^="my"]')
            * $('a[title$="blog"]')
            * $('a[href*="ziip"]')包含
            * $('a[ref][href][title$="blog"]')  并且
        4. Child Filters 子元素过滤器

            * $('li:nth-child(even)')
            * $('li:nth-child(odd)')
            * $('li:nth-child(2)')
            * $('li:nth-chilid(2n)')
            * $('li:first-child')
            * $('li:last-child')
            * $('code:only-child')
        5. Form 表单选择器

            * $(':input')
            * $(':text')
            * $(':password')
            * $(':radio')
            * $(':checkbox')
            * $(':submit')
            * $(':image')
            * $(':reset')
            * $(':button')
            * $(':file')
        6. Form Filter 表单过滤器

            * $('input:enabled')
            * $(':disabled')
            * $(':checked')
            * $(':selected')
    4. DOM操作

        1. 获取设定内容 

            * text() 转换<>
            * html() 包含<>
            * val() 获取 input框的value值
        2. 获取设定属性

            * attr()
            * removeAttr()
        3. 获取设定 CSS Class

            * addClass()
            * removeClass()
            * hasClass()
        4. 获取设定 CSS style

            1. css()
        5. append() and prepend()

            * append()
            * prepend()
        6. appendTo() and prependTo()

            * appendTo()
            * prependTo()
        7. before() and after()

            * before()
            * after()
        8. insertBefore() and insertAfter()

            * insertBefore()
            * insertAfter()
        9. remove(),empty() and detach()

            * remove()
            * empty()
            * detach()
        10. wrap()

            * wrap()
        11. replaceWith() and replaceAll()

            * replaceWith()
            * replaceAll
        12. clone() and Cloning Event Handlers and Data

            * clone()
    5. 事件处理 

        1. jQuery Event Handlers

            * ready()
            * blur()
            * focus()
            * click()
            * dbclick()
            * mousedown()
            * mouseup()
            * mousemove()
            * mouseover()
            * mouseout
            * keydown()
            * keypress()
            * keyup
            * change()
            * submit()
        2. jQuery Event Object Properties

            1. event.altkey
            2. event.ctrlKey
            3. event.data
            4. event.keyCode
            5. event.pageX
            6. event.pageY
            7. event.screenX
            8. event.screenY
            9. event.shiftKey
            10. event.target
            11. event.timeStamp
            12. event.type
    6. 动画效果

        1. 基本动画

            * show
            * hide
            * toggle
        2. 预置动画

            * slideDown(speed,[callback])
            * slideUp(speed,[callback] )
            * slideToggle(speed,[callback] )
            * fadeIn(speed,[callback] )
            * fadeOut(speed,[callback] )
            * fadeTo(opacity,[callback] )
        3. 自定义动画

            * animate( params,[duration,[easing],[callback]]
            * animate(params,options)
            * stop([clearQueue],[gotoEnd])
    7. Ajax异步请求

        1. load( url,[data],[callback])
        2. $.get(url,[data],[callback]]
        3. $.post(url,[data],[callback()]
        4. $.getScript(url, [callback])载入并执行一个外部js文件
        5. jQuery Ajax事件
    8. jQuery UI 入门
    9. jQuery编程规范
    10. 实践课：用jQuery和Local storage写一个Todo list网页应用
