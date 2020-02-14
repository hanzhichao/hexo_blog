---
layout: post
title: DjangoWeb开发指南读书笔记
comments: true
toc: true
date: 2018-02-01 20:01:43
updated: 2018-02-01 20:01:43
categories:
tags: Django
---
第5章 URL Http 机制和视图

5.1 URL

    * urlconf
用url替换元组 kwargs参数 和 name
    * 使用多过patterns对象，（1.10以弃用patterns）
urlpatterns+= patterns（）
    * 用include来包含其他url文件，（可以指定namespace）


5.2 http建模:请求，响应和中间件

    * 请求对象----request.GET  request.POST
    * cookies和会话，（sessions）----request.cookies  request.session
    * 其他服务器变量----request.

        * path 资源路径/blog/2017/11/04/
        * method 请求方法，GET,PUT,POST,DELETE 
        * FILES 上传的文件
        * Meta 请求变量
        * user django的认证用户，激活django的认证机制时才会出现
        * raw_post_data post原始数据
    * 响应对象

        * response=HttpResponse()
        * response.write()
        * response["content-type"]="text/csv"
    * 中间件
中间件就是一个python类，他实现了一个特定的接口，定义了一些名为process_request或是process_view这样的方法----按settings.py中有序

        * 请求中间件---先逐个处理请求中间件，最后再处理views本身
        * 响应中间件---接受一个request和response重新返回一个新的httpresponse


5.3视图与逻辑

    * 就是python函数
    * 通用视图
    * 半通用视图
    * 自定义视图

        * render_to_response
        * http404
        * get_object_or_404
        * get_listor_404
    * 其他观察

        * views中使用，*args **kwargs



第6章 模板和表单处理

6.1 模板

    * 理解Contexts ---一个类似字典的Contexts对象，views向模板传递变量

        * 上下文处理器 TEMPLATE_CONTEXT_PROCESSORS
    * 模板语言语法

        * 单独命令 --{{ 变量 }}--变量输出
        * 模块级命令{% if %} {% endif %}
    * 模板过滤器
    * 标签 tag---- if,ifequal,block,include,extends

        * 可以结合过滤器
        * Blocks和Extends
        * 包含其他模板include

6.2表单

    * 定义表单

        * forms.Form---简单表单，不关联model
        * 基于模型的表单---forms.ModelForm---关联Model
class PersonForm(forms.ModelForm):
    class Meta:
        model = Person
        * 保存ModelForm ---form.save(),form.save(commet=False)

            * 区别于Model---fields和exclude----两种只能用一个，被忽略的字段必须null=True（运行为null）
        * 继承表单，多重继承---使用fields或exculde来限制变量
    * 填写表单

        * 绑定的 未绑定的
        * requests.POST.copy()
        * initial制定默认的值---last = forms.CharField(max_length=100, required=True,initial='Smith')
    * 显示表单 可以通过auto_id 和label_suffix指定 input的id和文本

        * pf = TestForm(auto_id=False, label_suffix='')
        * pf = TestForm(auto_id=%_id, label_suffix="？")
        * 显示全部表单

            * form.as_table---使用<table>标签
            * from.as_p---使用p标签
            * form.as_ul---使用ul标签
            * 可以继承django.forms.util.ErrorList 来自定义错误列表的显示方式
        * 逐个显示表单

            * Widget---控制显示
            * middle=forms.CHarField(max_length=100, widget=forms.TextInput(attrs={'size':3})
            * 重写一个变量的Widget


class LargeTextareaWidget(forms.Textarea):
    def __init__(self, *args, **kwargs):
        kwargs.setdefault('attrs',{}).update({'rows':40,'cols':100})
        super(LargeTextareaWidget, self).__init__(*args,**kwargs)

class LargeTextarea(forms.Field):
    widget = LargeTextareaWidget

#froms.py
class ContentForm(forms.Form):
    name=forms.CharField()
    text = LargeTextarea()

