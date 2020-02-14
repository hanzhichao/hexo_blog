---
layout: post
title: Django开发博客-版本二
comments: true
toc: true
date: 2018-02-01 19:37:45
updated: 2018-02-01 19:37:45
categories:
tags:
---
# setting.py
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, "static"),
]
AUTH_USER_MODEL = 'blog.User'
# urls.py
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^blog/', include('blog.urls', namespace='blog')),

]

# blog/models.py
# coding:utf8
from django.db import models
from django.contrib.auth.models import AbstractUser


class Category(models.Model):
    names = models.CharField('类型名称', max_length=20)
    desc = models.TextField('分类描述', blank=True, null=True)

    def __unicode__(self):
        return self.names
    

class User(AbstractUser):
    pass


class Post(models.Model):
    POST_STATUS =(
        ('P', '已发布'),
        ('D', '已删除'),
        ('E', '编辑中')
    )

    title = models.CharField('标题', max_length=150)
    body = models.TextField('正文')
    create_time = models.DateTimeField('创建时间', auto_now_add=True)
    modify_time = models.DateTimeField('最后一次修改', auto_now=True)
    status = models.CharField('文章状态', max_length=1, choices=POST_STATUS)
    views = models.PositiveIntegerField('浏览量', default=0)
    likes = models.PositiveIntegerField('喜欢', default=0)
    praises = models.PositiveIntegerField('点赞', default=0)
    category = models.ManyToManyField('Category', blank=True)
    user = models.ForeignKey('User', verbose_name='作者')

    def __unicode__(self):
        return self.title

    class Meta:
        ordering = ('-create_time',)


class BlogComment(models.Model):
    user = models.ForeignKey('User', verbose_name='用户')
    post = models.ForeignKey('Post',verbose_name='文章')
    comment = models.TextField('评论内容', blank=True)
    comment_time = models.DateTimeField('评论时间', auto_now_add=True)

    class Meta:
        ordering = ('-comment_time',)


# blog/admin.py
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from blog.models import Post, Category, BlogComment, User


class BlogUserAdmin(UserAdmin):
    pass


class BlogAdmin(admin.ModelAdmin):
    list_display = ('title', 'create_time')

admin.site.register(Post, BlogAdmin)
admin.site.register(Category)
admin.site.register(BlogComment)
admin.site.register(User, BlogUserAdmin)
# blog/views.py
from django.shortcuts import render, render_to_response, get_object_or_404
from django.http import HttpResponse, HttpResponseRedirect
from django.views.decorators.csrf import csrf_exempt
# from django.contrib.auth.models import User
from django.contrib import auth
from django.contrib.auth import get_user_model
import json
from blog.models import *

User = get_user_model()


def index(request):
    categories = Category.objects.all()
    articles = Post.objects.all()
    return render_to_response('blog/index.html', locals())


@ csrf_exempt
def login(request):
    if request.method == "POST":
        username = request.POST.get('username')
        password = request.POST.get('password')
        userquery = User.objects.filter(username=username)
        for user in userquery:
            if user is not None and user.check_password(password):
                user.backend = 'django.contrib.auth.backends.ModelBackend'
                auth.login(request, user)
                return HttpResponseRedirect("../")
        return HttpResponse(json.dumps(&#123;'error': 'User not exist'&#125;))
    return render_to_response("blog/login.html")


def logout(request):
    auth.logout(request)
    return HttpResponseRedirect("../")


def register(request):
    return render_to_response("blog/register.html")
    return HttpResponseRedirect("../")


def edit(request):
    pass


def category(request, slug):
    cur_category = get_object_or_404(names=slug)
    return render_to_response()


def article(request, slug):
    pass
# blog/urls.py
from django.conf.urls import url
from blog.views import *
urlpatterns = [
    url(r'^$', index, name='index'),
    url(r'^login/$', login, name='login'),
    url(r'^logout/$', logout, name='logout'),
    url(r'^register/$', register, name='register'),
    url(r'^edit/$', edit, name='edit'),
    url(r'^category/(?P<slug>[-\w]+)/$', category, name='category'),
    url(r'^article/(?P<slug>[-\w]+)/$', article, name='category'),
]
# base.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>&#123;% block title %&#125;&#123;% endblock %&#125;</title>
    &#123;% load static %&#125;
    <link rel="stylesheet" type="text/css" href="&#123;% static 'blog/css/bootstrap.min.css' %&#125;">
    <script type="text/javascript" src="&#123;% static 'blog/js/jquery.min.js' %&#125;"></script>
    <script type="text/javascript" src="&#123;% static 'blog/js/bootstrap.min.js' %&#125;"></script>
</head>
<body>
&#123;% block header %&#125;
    <div class="navbar navbar-default">
        <div class="container-fluid">
            <div class="nav-header">
                <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar-collapse" aria-expanded="false">
                </button>
                <div class="navbar-brand">
                    <a href="&#123;% url 'blog:index' %&#125;">StudyBlog</a>
                </div>
            </div>
            <div class="collapse navbar-collapse" id="navbar-collapse">
                <nav class="nav navbar-nav">
                    <li class="active"><a href="#">发现</a></li>
                    <li><a href="#">关注</a></li>
                    <li><a href="#">消息</a></li>
                </nav>
                <form class="navbar-form navbar-left">
                    <div class="form-group">
                        <input type="text" class="form-control" placeholder="搜索...">
                    </div>
                    <button type="submit" class="btn btn-default">搜索</button>
                </form>
                <div &#123;% if request.user.is_authenticated %&#125; style="display:none"&#123;% endif %&#125;>
                    <form class="navbar-form navbar-right">
                        <a class="btn btn-primary" href="&#123;% url 'blog:login' %&#125;">登录</a>
                        <a class="btn btn-primary" href="&#123;% url 'blog:register' %&#125;">注册</a>
                    </form>
                </div>
                <div &#123;% if not request.user.is_authenticated %&#125; style="display: none"&#123;% endif %&#125;>
                    <form class="navbar-form navbar-right">
                        <a class="btn btn-primary" href="&#123;% url 'blog:edit' %&#125;">写文章</a>
                    </form>
                    <ul class="nav navbar-nav navbar-right">
                        <li class="dropdown">
                            <a href="#" class="dropdown-toggle" data-toggle="dropdown" role="button" aria-haspopup="true" aria-expanded="false">&#123;&#123; request.user.username &#125;&#125;<span class="caret"></span></a>
                        <ul class="dropdown-menu">
                            <li><a class="item" href="#">我的主页</a></li>
                            <li><a class="item" href="#">我的评论</a></li>
                            <li><a class="item" href="&#123;% url 'blog:logout' %&#125;">退出</a></li>
                        </ul>
                        <li>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
&#123;% endblock %&#125;
&#123;% block content %&#125;
&#123;% endblock %&#125;
<footer>
    &#123;% block footer %&#125;
    &#123;% endblock %&#125;
</footer>
</body>
</html>
#index.html
&#123;% extends 'blog/base.html' %&#125;
&#123;% block title %&#125;主页&#123;% endblock %&#125;
&#123;% block content %&#125;
<div class="row-fluid">
    <div class="col-md-2">
        <nav class="nav nav-list">
            &#123;% for category in categories %&#125;
            <li><a href="&#123;% url 'blog:category' category.names %&#125;">&#123;&#123; category.names &#125;&#125;</a></li>
            &#123;% endfor %&#125;
        </nav>
    </div>
    <div class="col-md-10">
        &#123;% for article in articles %&#125;
            <h1>&#123;&#123; article.title &#125;&#125;</h1>
            &#123;&#123; article.body|safe&#125;&#125;
        &#123;% endfor %&#125;
    </div>
</div>
&#123;% endblock %&#125;
#login.html
&#123;% extends 'blog/base.html' %&#125;
&#123;% block title %&#125;登录&#123;% endblock %&#125;
&#123;% block header %&#125;&#123;% endblock %&#125;
&#123;% block content %&#125;
    <h1>&nbsp;</h1>
    <div class="row">
        <div class="col-md-4 col-md-offset-4">
            <div class="panel panel-info">
                <div class="panel-heading">
                    <div class="panel-title"><h1>登录</h1></div>
                    <div class="panel-body">
                        <form class="form" method="post">
                            <div class="form-group">
                                <div class="input-group">
                                    <span class="input-group-addon"><span
                                            class="glyphicon glyphicon-user"></span></span>
                                    <input type="text" class="form-control" placeholder="用户名"
                                           aria-describedby="basic-addon1" name="username">
                                </div>
                            </div>
                            <div class="form-group">
                                <div class="input-group">
                                    <span class="input-group-addon"><span class="glyphicon glyphicon-eye-close"></span></span>
                                    <input type="password" class="form-control" placeholder="密码" name="password">
                                </div>
                            </div>
                            <div class="form-group">
                                <button type="submit" class="btn btn-primary form-control">登录</button>
                            </div>
                        </form>
                    </div>
                    <div class="panel-footer">请登录...</div>
                </div>
            </div>
        </div>
    </div>
&#123;% endblock %&#125;

#register.html
&#123;% extends 'blog/base.html' %&#125;
&#123;% block title %&#125;注册&#123;% endblock %&#125;
&#123;% block header %&#125;&#123;% endblock %&#125;
&#123;% block content %&#125;
    <h1>&nbsp;</h1>
    <div class="row">
        <div class="col-md-4 col-md-offset-4">
            <div class="panel panel-info">
                <div class="panel-heading">
                    <div class="panel-title"><h1>注册</h1></div>
                    <div class="panel-body">
                        <form class="form" method="post">
                            <div class="form-group">
                                <div class="input-group">
                                    <span class="input-group-addon"><span
                                            class="glyphicon glyphicon-user"></span></span>
                                    <input type="text" class="form-control" placeholder="用户名" name="username">
                                </div>
                            </div>
                            <div class="form-group">
                                <div class="input-group">
                                <span class="input-group-addon"><span
                                        class="glyphicon glyphicon-envelope"></span></span>
                                    <input type="email" class="form-control" placeholder="邮箱" name="email">
                                </div>
                            </div>
                            <div class="form-group">
                                <div class="input-group">
                                <span class="input-group-addon"><span
                                        class="glyphicon glyphicon-eye-close"></span></span>
                                    <input type="password" class="form-control" placeholder="密码" name="password">
                                </div>
                            </div>
                            <div class="form-group">
                                <div class="input-group">
                                <span class="input-group-addon"><span
                                        class="glyphicon glyphicon-eye-close"></span></span>
                                    <input type="password" class="form-control" placeholder="重复密码" name="repassword">
                                </div>
                            </div>
                            <div class="form-group">
                                <button type="submit" class="btn btn-primary form-control">注册</button>
                            </div>
                        </form>
                    </div>
                    <div class="panel-footer">请注册...</div>
                </div>
            </div>
    </div>
&#123;% endblock %&#125;

