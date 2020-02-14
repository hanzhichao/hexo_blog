---
layout: post
title: Django开发简单CMS-版本二
comments: true
toc: true
date: 2018-02-01 19:49:28
updated: 2018-02-01 19:49:28
categories:
tags: Django
---
Django 1.8.3

## 项目级
### setting.py
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
)
### urls.py
from django.conf.urls import include, url
from django.contrib import admin
urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^cms/', include('cms.urls')),
]

## 应用级
### urls.py
from django.conf.urls import patterns, url
from cms.views import home, category, article, search
urlpatterns = [
    url(r'^$', home, name='cms-home'),
    url(r'^category/(?P<slug>[-\w]+)/$', category, name='cms-category'),
    url(r'^article/(?P<slug>[-\w]+)/$', article, name='cms-article'),
    url(r'^search/$', search, name="cms-search"),
]
### views.py
# coding=utf-8
from cms.models import Article, Category
from django.shortcuts import render_to_response, get_object_or_404
from django.db.models import Q
def home(request):  # catrgory list
    categories = Category.objects.all()
    return render_to_response("cms/index.html", &#123;'categories': categories&#125;)
def category(request, slug):  # article list
    """Given a category slug,display all items in a category."""
    categories = Category.objects.all()
    cur_category = get_object_or_404(Category, slug=slug)
    article_list = Article.objects.filter(category=cur_category)
    heading = cur_category.label
    return render_to_response("cms/category.html", locals())  # 返回所有本地变量
def article(request, slug):  # article detail # 怎么设计的不同分类可重复？
    categories = Category.objects.all()
    cur_article = get_object_or_404(Article, slug=slug)
    return render_to_response("cms/article.html", locals())  # 返回所有本地变量
def search(request):
    """
    Return a list of stories that match the provided search term
    in either the title or the main content.
    """
    categories = Category.objects.all()
    if 'q' in request.GET:
        term = request.GET('q')
        story_list = Article.objects.filter(Q(title__contains=term) | Q(markdown_content__contains=term))
        heading = "Search results"
    return render_to_response("cms/category.html", locals())

### modles.py
# coding=utf-8
from __future__ import unicode_literals
from django.db import models
import datetime
from django.db.models import permalink
from django.contrib.auth.models import User
from django.utils.encoding import python_2_unicode_compatible
from markdown import markdown
from django.contrib import admin
VIEWABLE_STATUS = [3, 4]
# @python_2_unicode_compatible  # 向后兼容django版本
class Category(models.Model):
    """文章分类"""
    label = models.CharField(blank=True, max_length=50)
    slug = models.SlugField()
    class Meta:
        verbose_name_plural = '分类'
    @permalink
    def get_absolute_url(self):  # 获取reverse后的链接
        # return ('cms-article', (), &#123;'slug': self.slug&#125;)
        return 'cms-category', (), &#123;'slug': self.slug&#125;  # 尝试一下是否可以
    def __unicode__(self):
        return self.label
# @python_2_unicode_compatible  # 向后兼容django版本，可以使用 def __str__()
class Article(models.Model):
    """ 文章页面 """
    STATUS_CHOICES = &#123;
        (1, '草稿'),
        (2, '待审批'),
        (3, '已发布'),
        (4, '存档')
    &#125;
    title = models.CharField('标题', max_length=256)
    slug = models.SlugField('网址', max_length=256, db_index=True)  # 建立数据库索引
    category = models.ForeignKey(Category)
    markdown_content = models.TextField()
    html_content = models.TextField(editable=False)  # invisible
    # author = models.ForeignKey(User, blank=True, null=True)
    author = models.ForeignKey('auth.User', blank=True, null=True, verbose_name='作者')
    status = models.IntegerField('状态', choices=STATUS_CHOICES, default=1)
    # created = models.DateTimeField(default=datetime.datetime.now)  # 可以杜撰
    created = models.DateTimeField('创建时间', auto_now_add=True)  # 第一次创建更改
    # modified = models.DateTimeField(default=datetime.datetime.now) # 可以杜撰
    modified = models.DateTimeField('最后修改时间', auto_now=True)  # 每次保存时间
    class Meta:
        ordering = ['modified']
        verbose_name = '文章'  # 单数 中文下显示
        verbose_name_plural = '文章'  # 复数，中文下显示
    @permalink
    def get_absolute_url(self):  # 获取reverse后的链接
        # return ('cms-article', (), &#123;'slug': self.slug&#125;)
        return 'cms-article', (), &#123;'slug': self.slug&#125;  # 尝试一下是否可以
    # def __unicode__(self):  # 在Python3中用 __str__ 代替 __unicode__
    #     return self.title
    def save(self, *args, **kws):  # 转换markdown内容为html内容
        self.html_content = markdown(self.markdown_content)
        self.modified = datetime.datetime.now()
        super(Article, self).save()
class ViewableManager(models.Manager):
    def get_query_set(self):
        default_queryset = super(ViewableManager, self).get_query_set()
        return default_queryset.filter(status__in=VIEWABLE_STATUS)

### admin.py
from django.contrib import admin
from cms.models import Article, Category


class ArticleAdmin(admin.ModelAdmin):
    list_display = ('title', 'author', 'status', 'created', 'modified')
    search_fields = ('title', 'content')
    list_filter = ('status', 'author', 'category', 'created', 'modified')
    # prepopulated_fields = &#123;'slug': ('title',)&#125;  # 同步slug与title相同


class CategoryAdmin(admin.ModelAdmin):
    pass
    # prepopulated_fields = &#123;'slug': ('label',)&#125;  # 同步slug与title相同

admin.site.register(Article, ArticleAdmin)
admin.site.register(Category, CategoryAdmin)

## 模板
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    &#123;% load staticfiles %&#125;
    <link rel="stylesheet" href="&#123;% static 'css/bootstrap.min.css' %&#125;">
    <title>&#123;% block title %&#125;&#123;% endblock %&#125;</title>
</head>
<body>
    <div class="container" style="margin-top: 10px;">
        <nav class="navbar navbar-default">
            <div class="nav-header">
                <div class="navbar-brand"><a href="#">MY-CMS</a></div>
            </div>
            <ul class="nav navbar-nav">
                <li><a href="&#123;% url 'cms-home' %&#125;">首页</a> </li>
                &#123;% for category in categories %&#125;
                    <li><a href="&#123;&#123; category.get_absolute_url &#125;&#125;">&#123;&#123; category &#125;&#125;</a></li>
                &#123;% endfor %&#125;
            </ul>
            <ul class="nav navbar-nav navbar-right" style="margin-right: 10px;">
                <li><a href="../admin/" class="navbar-link">登录</a></li>
            </ul>
            <form class="navbar-form" action="&#123;% url 'cms-search' %&#125;" method="get">  <!--1.5之后要用引号-->
                <input class="form-control" type="text" name="q" placeholder="Search">
            </form>
        </nav>
    </div>
    <div class="container">
        &#123;% block content %&#125;
        &#123;% endblock %&#125;
    </div>
</body>
</html>
### index.html
&#123;% extends "cms/base.html" %&#125;
&#123;% block title %&#125;&#123;&#123; story.title &#125;&#125;&#123;% endblock %&#125;
&#123;% block content %&#125;
    &#123;% if heading %&#125;
        <h1>&#123;&#123; heading &#125;&#125;</h1>
    &#123;% endif %&#125;
    <div class="jumbotron">
        <h1>Welcome,Friend</h1>
        <p>
           This is a simple hero unit,
            a simple jumbotron-style component for calling extra attention
            to featured content or information.
        </p>
        <p><a class="btn btn-primary btn-lg" href="#" role="button">Learn more</a></p>
    </div>
    <div class="page-header">
        <h1>首页<small>（分类列表）</small></h1>
    </div>
    <ol class="breadcrumb">
    <li><a href="&#123;% url 'cms-home' %&#125;">首页</a> </li>
</ol>
    <div class="page">
         <ul>
        &#123;% for category in categories %&#125;
            <li><a href="&#123;&#123; category.get_absolute_url &#125;&#125;">&#123;&#123; category.label &#125;&#125;</a> </li>
        &#123;% endfor %&#125;
    </ul>
    </div>
&#123;% endblock %&#125;
### article.html
&#123;% extends "cms/base.html" %&#125;
&#123;% block title %&#125;&#123;&#123; article.title &#125;&#125;&#123;% endblock %&#125;
&#123;% block content %&#125;
<div class="page-header">
    <h1>&#123;&#123; cur_article.title &#125;&#125;</h1>
</div>
<ol class="breadcrumb">
    <li><a href="&#123;% url 'cms-home' %&#125;">首页</a> </li>
    <li><a href="&#123;% url 'cms-category' cur_article.category.slug %&#125;">&#123;&#123; cur_article.category &#125;&#125;</a> </li>
    <li><a href="&#123;&#123; cur_article.get_absolute_url &#125;&#125;">&#123;&#123; cur_article.title &#125;&#125;</a> </li>
</ol>
<div class="page">
    &#123;&#123; cur_article.html_content|safe &#125;&#125;
    <p class="small">最后更新： &#123;&#123; cur_article.modified &#125;&#125;</p>
</div>
&#123;% endblock %&#125;
###category.html
&#123;% extends "cms/base.html" %&#125;
&#123;% block title %&#125;&#123;&#123; story.title &#125;&#125;&#123;% endblock %&#125;
&#123;% block content %&#125;
    &#123;% if heading %&#125;
        <div class="page-header">
        <h1>&#123;&#123; heading &#125;&#125;</h1>
        </div>
    &#123;% endif %&#125;
<ol class="breadcrumb">
<li><a href="&#123;% url 'cms-home' %&#125;">首页</a> </li>
<li><a href="&#123;&#123; cur_category.get_absolute_url &#125;&#125;">&#123;&#123; cur_category &#125;&#125;</a> </li>
</ol>
<div class="page">
    <ul>
        &#123;% for article in article_list %&#125;
            <li><a href="&#123;&#123; article.get_absolute_url &#125;&#125;">&#123;&#123; article.title &#125;&#125;</a> </li>
        &#123;% endfor %&#125;
    </ul>
</div>
&#123;% endblock %&#125;
