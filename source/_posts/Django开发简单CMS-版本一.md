---
layout: post
title: Django开发简单CMS
comments: true
toc: true
date: 2018-02-01 19:36:09
updated: 2018-02-01 19:36:09
categories:
tags: Django
---
根据自强学堂教程编写

# requirements.txt
Django==1.8.3
DjangoUeditor
# README.md
## A CMS PROJECT by Django
[参考教程](http://code.ziqiangxuetang.com/django/django-cms-develop.html)
>modified DjangoUeditor/urls.py

#coding:utf-8
from django import VERSION
if VERSION[0:2]>(1,9):
    from django.conf.urls import url
elif VERSION[0:2]>(1,3):
    from django.conf.urls import patterns, url
else:
    from django.conf.urls.defaults import patterns, url
from views import get_ueditor_controller
if VERSION[0:2]>(1,9):
    urlpatterns = [
        url(r'^controller/$',get_ueditor_controller)
    ]
else:
    urlpatterns = patterns('',
        url(r'^controller/$',get_ueditor_controller)
    )
urls.py
from django.conf.urls import include, url
from django.contrib import admin
# from DjangoUeditor import urls as DjangoUeditor_urls
from django.conf import settings


urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^ueditor/', include('DjangoUeditor.urls')),
    # url(r'ueditor/', include(DjangoUeditor_urls)),
    url(r'^$', 'cms.views.index', name='index'),

    url(r'^column/(?P<column_slug>[^/]+)/$', 'cms.views.column_detail', name='column'),
    url(r'^article/(?P<article_slug>[^/]+)/$', 'cms.views.article_detail', name='article'),
    url(r'^static/(?P<path>.*)$', ' django.views.static.serve', &#123;'document_root': settings.STATIC_ROOT&#125;),
    url(r'^admin/static/(?P<path>.*)$', 'django.views.static.serve',
        &#123;'document_root': settings.STATIC_ROOT&#125;),
]
if settings.DEBUG:
    from django.conf.urls.static import static
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
# settings.py
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'DjangoUeditor',
    'cms',
)

TEMPLATES = [
    &#123;
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        'APP_DIRS': True,
        'OPTIONS': &#123;
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        &#125;,
    &#125;,
]


# LANGUAGE_CODE = 'en-us'
LANGUAGE_CODE = 'zh-hans'

# TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/Shanghai'

USE_I18N = True

USE_L10N = True

USE_TZ = True


# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.8/howto/static-files/

# Static files (CSS, JavaScript, Images)
STATIC_URL = 'static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static').replace('\\','/')

#STATICFILES_DIRS =(
#    os.path.join(BASE_DIR, "common_static")
#)

# upload folder
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
#cms/models.py
# _*_ coding: utf-8 _*_
from __future__ import unicode_literals

from django.db import models
from django.utils.encoding import python_2_unicode_compatible
from DjangoUeditor.models import UEditorField
from django.core.urlresolvers import reverse


@python_2_unicode_compatible
class Column(models.Model):
    pre_column = models.ForeignKey('self', verbose_name='上级栏目', blank=True, null=True)
    name = models.CharField('栏目名称', max_length=256)
    slug = models.CharField('栏目网址', max_length=256, db_index=True, unique=True)
    intro = models.TextField('栏目简介', default='')
    thumbnail = models.ImageField('缩略图', blank=True, null=True)

    keywords = models.CharField('关键字', max_length=80, blank=True, null=True)
    description = models.TextField('描述', blank=True, null=True)

    def __str__(self):
        return self.name

    class Meta:
        verbose_name = '栏目'
        verbose_name_plural = '栏目'
        ordering = ['name']

    def get_absolute_url(self):
        return reverse('column', args=(self.slug,))


@python_2_unicode_compatible
class Article(models.Model):
    STATUS_CHOICES = (
        ('d', "Draft"),
        ('p', 'Published'),
    )

    title = models.CharField('标题', max_length=256)
    keywords = models.CharField('关键字', max_length=80, blank=True, null=True)
    slug = models.CharField('网址', max_length=256,db_index=True, unique=True)
    column = models.ForeignKey(Column, verbose_name='归属栏目')
    author = models.ForeignKey('auth.User',blank=True, null=True, verbose_name='作者')
    thumbnail = models.ImageField('缩略图', blank=True, null=True)

    abstract = models.TextField('摘要', max_length=54, blank=True, null=True,
                                help_text="可选,若为空将摘取正文的前54个字符")
    description = models.TextField('描述', blank=True, null=True)
    # content = models.TextField("内容", blank=True, null=True)
    content = UEditorField('内容', height=300, width=1000, default=u'', blank=True,
                           imagePath="uploads/images/",
                           toolbars='besttome', filePath='uploads/files/')

    created_time = models.DateTimeField('创建时间', auto_now_add=True)
    last_modified_time = models.DateTimeField('修改时间', auto_now=True)
    views = models.PositiveIntegerField('浏览量', default=0)
    likes = models.PositiveIntegerField('点赞数', default=0)
    topped = models.BooleanField('置顶', default=False)

    def __str__(self):
        return self.title

    class Meta:
        verbose_name = '文章'
        verbose_name_plural = '文章'
        ordering = ['-last_modified_time']

    def get_absolute_url(self):
        return reverse('article', args=(self.slug,))
#cms/admin.py
from django.contrib import admin
from .models import Column, Article


class ColumnAdmin(admin.ModelAdmin):
    list_display = ('name', 'pre_column', 'intro')


class ArticleAdmin(admin.ModelAdmin):
    list_display = ('title', 'column', 'last_modified_time')

admin.site.register(Column, ColumnAdmin)
admin.site.register(Article, ArticleAdmin)

#cms/views.py
# _*_ coding: utf-8 _*_
from django.shortcuts import render
from django.http import HttpResponse
from .models import Column, Article
from unicms import settings


def index(request):
    columns = Column.objects.all()
    # return HttpResponse(settings.STATIC_ROOT)
    return render(request, 'index.html', &#123;'columns': columns&#125;)


def column_detail(request, column_slug):
    # return HttpResponse('column slug: ' + column_slug)
    column = Column.objects.filter(slug=column_slug)
    return render(request, 'single.html', &#123;'column': column&#125;)


def article_detail(request, article_slug):
    # return HttpResponse('article slug: ' + article_slug)
    article = Article.objects.filter(slug=article_slug)[0]
    return render(request, 'single.html', &#123;'article':article&#125;)
