---
layout: post
title: Javascript初步学习笔记
comments: true
toc: true
date: 2018-02-01 19:56:12
updated: 2018-02-01 19:56:12
categories:
tags: 前端
---
    1. 概述

        1. 特点

            1. 大小写敏感
            2. 使用；结束，可以一行写多个语句，不写在行尾自动添加
        2. 引用方式

            1. 嵌入html<script></script>中直接编写
            2. 引入js文件<script src='**.js'></script>引入外部js文件
            3. 事件或链接触发，作为某个原始的事件属性值或超链接href属性值
            4. 代码屏蔽
<script type='text/javascript>'
<!--
js代码
//-->
</script>
        3. Debug方法

            1. alert();  --转换为数值
            2. document.write();  --转换为字符
    2. 变量

        1. 变量声明 var a=1,b=-2.3,c='hello',d=true,e=null,f=undefined---未声明的变量为undefined
        2. 变量类型

            1. 数值 整形（10/8/16进制 123,0123,0x123） 浮点型（1.2  0.8e3 1.8E-3）---NaN 表示Not a Number,非数值型,Infine无穷大（超过取值范围）
            2. 字符串 + 字符串连接
            3. 布尔类型 true,false
            4. 对象型 数组 undifined ......
        3. 类型转换

            1. 隐式转换

                1. 转数值 ‘123' ->
            2. 显示转换
            3. 转换规则

                1. 转数值  '123' ->123,true->1,false->0,null->0,undefined->NaN, '123king'->NaN(用PasteInt转换为123）
                2. 转字符串
                3. 转布尔 null,0,-0,0.0,'',undefined -> false,其他->true
        4. 变量作用范围

            1. 全局变量

                1. 函数外var声明的变量
                2. 函数内直接赋值的变量
            2. 局部变量

                1. 函数内var声明的变量
            3. 注意：函数中尽量使用局部变量（用var声明），避免操作导致全局变量的变化
    3. 运算符及优先级
    4. 分支
    5. 循环
    6. 函数

        1. 自定义函数
        2. 全局函数
        3. 特殊函数

            1. 匿名函数 var a=function(x,y){return x+y;}
            2. 回调函数 function a(x,y,callback){ x=callback(x);y=callback(y);return x+y;} function callback(x){return x+1;}
            3. 自调函数 ( function(x,y){ return x+y; } )(1,2); 不产生全局变量，只能执行一次
            4. 私有函数 function a(x){ function b(z){ return z+1;}; return b(x);}  私有函数外部不能执行
            5. 返回函数 var test=function a(){ function b(z){return z+1;} return b;}; test(3);
            6. 重新函数 (fanction a(){ alert('aaa');a=function(){alert('bbb');}})(); a(); 执行一次后变更
var a=function(){
     function setUp(){
         alert('初始化'); 
     }
     function test(){
         alert('测试') 
     }
     setUp();
     return test;
}();
a();
            7. 闭包
    7. 对象

        1. 自定义对象
        2. 内建对象

            1. 数据封装对象

                1. Object对象
                2. Object.prototype
                3. Number
                4. Boolean
                5. String
                6. Array
                7. Function
            2. 工具类对象

                1. Date
                2. Math

                    1. 属性

                        1. Math.E 欧拉常数
                        2. Math.LN2 2的对数
                        3. Math.LN10
                        4. Math.LOG2E
                        5. Math.LOG10E
                        6. Math.PI
                        7. Math.SQRT1_2 1/2的平方根
                        8. Math.SQRT2
                    2. 方法

                        1. Math.abs(x)
                        2. Math.ceil(x)向上取整
                        3. Math.floor(x)
                        4. Math.round(x)四舍五入
                        5. Math.pow(x,y)x的y次幂
                        6. Math.sqrt(x)平方根
                        7. ......
                        8. 

            3. 错误对象

                1. Error对象
