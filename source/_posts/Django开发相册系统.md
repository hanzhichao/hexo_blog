---
layout: post
title: Django开发相册系统
comments: true
toc: true
date: 2018-02-01 19:42:07
updated: 2018-02-01 19:42:07
categories:
tags: Django
---

## 项目级

### settings.py
MEDIA_ROOT = os.path.join(BASE_DIR, 'media').replace('\\', '/')

# DRY(do not repeat yourself) URL
ROOT_URL = '/gallery/'
LOGIN_URL = ROOT_URL + 'login/'
MEDIA_URL = ROOT_URL + 'media/'
ADMIN_MEDIA_PREFIX = MEDIA_ROOT + 'admin/'

###urls.py
from django.conf.urls import include, url, patterns
from book_gallery.settings import ROOT_URL

urlpatterns = [
    url(r'^%s' % ROOT_URL[1:], include('book_gallery.real_urls')),
]

### real_urls.py
from django.conf.urls import patterns, include, url
from django.contrib import admin
from book_gallery.settings import ROOT_URL
def root_url_processor(request):
    return &#123;'ROOT_URL': ROOT_URL&#125;
urlpatterns = patterns('',
                       url(r'^admin/', include(admin.site.urls), name=admin),
                       url(r'^', include('gallery.urls')),
                       )

## 应用级
### urls.py
from django.conf.urls import patterns, include, url
from book_gallery import settings
from django.contrib import admin
from gallery.views import index, ItemList, ItemDetail, PhotosDetail
urlpatterns = [
    url(r'^admin/', include(admin.site.urls), name='admin'),
   url(r'^$', index, name='index'),
   url(r'items/$', ItemList.as_view(), name='item_list'),
   url(r'items/(?P<object_id>\d+)/$', ItemDetail.as_view(), name='item_detail'),
   url(r'photos/(?P<object_id>\d+)/$', PhotosDetail.as_view(), name='photos_detail'),
    url(r'^media/(?P<path>.*)$', 'django.views.static.serve', &#123;'document_root': settings.MEDIA_ROOT&#125;),
]
### fields.py
# coding:utf-8
from django.db.models.fields.files import ImageField, ImageFieldFile
from PIL import Image  # PIL: Python Image Library
import os
def _add_thumb(s):
    """
    Modifies a string(filename, URL) containing an image filename to insert
    '.thumb' before the file extension (which is changed to be '.jpg'
    """
    parts = s.split(".")
    parts.insert(-1, "thumb")
    if parts[-1].lower() not in ['jpg', 'jpeg']:
        parts[-1] = 'jpg'
    return ".".join(parts)  # join()用法 'sep'.join(seq)
class ThumbnailImageFieldFile(ImageFieldFile):
    def _get_thumb_path(self):
        return _add_thumb(self.path)
    thumb_path = property(_get_thumb_path)
    def _get_thumb_url(self):
        return _add_thumb(self.url)
    thumb_url = property(_get_thumb_url)
    def save(self, name, content, save=True):
        super(ThumbnailImageFieldFile, self).save(name, content, save)
        img = Image.open(self.path)
        img.thumbnail(
            (self.field.thumb_width, self.field.thumb_height),
            Image.ANTIALIAS
        )
        img.save(self.thumb_path, "JPEG")
    def delete(self, save=True):
        if os.path.exists(self.thumb_path):
            os.remove(self.thumb_path)
        super(ThumbnailImageFieldFile, self).delete(save)  # 删除原图
class ThumbnailImageFiled(ImageField):
    attr_class = ThumbnailImageFieldFile
    def __init__(self, thumb_width=400, thumb_height=320, *args, **kwargs):
        self.thumb_width = thumb_width
        self.thumb_height = thumb_height
        super(ThumbnailImageFiled, self).__init__(*args, **kwargs)
### views.py
# coding:utf-8
from django.shortcuts import render, render_to_response
from django.views.generic import ListView, DetailView
from .models import Item, Photo


def index(request):
    return render_to_response("gallery/index.html")


class ItemList(ListView):
    model = Item  # 默认取Item中全部数据
    context_object_name = 'item_list'  # 模板中使用的变量
    template_name = 'gallery/items_list.html'

    def get_context_data(self, **kwargs):  # 必须要有，不然无返回数据，可以在模型基础上新增返回数据
        item_list = super(ItemList, self).get_context_data(**kwargs)
        return item_list


class ItemDetail(DetailView):
    model = Item
    context_object_name = 'item'
    template_name = 'gallery/items_detail.html'
    pk_url_kwarg = 'object_id'  # 必须添加

    def get_context_data(self, **kwargs):  # 必须要有，不然无返回数据，可以在模型基础上新增返回数据
        item_detail = super(ItemDetail, self).get_context_data(**kwargs)
        return item_detail


class PhotosDetail(DetailView):
    model = Photo
    context_object_name = 'photo'
    template_name = 'gallery/photos_detail.html'
    pk_url_kwarg = 'object_id'

    def get_context_data(self, **kwargs):  # 必须要有，不然无返回数据，可以在模型基础上新增返回数据
        photos_detail = super(PhotosDetail, self).get_context_data(**kwargs)
        return photos_detail

### models.py
from django.db import models
from django.db.models import permalink
from gallery.fields import ThumbnailImageFiled


class Item(models.Model):
    name = models.CharField(max_length=250)
    description = models.TextField()

    class Meta:
        ordering = ['name']

    def __unicode__(self):
        return self.name

    @models.permalink
    def get_absolute_url(self):
        return 'item_detail', None, &#123;'object_id': self.id&#125;


class Photo(models.Model):
    item = models.ForeignKey(Item)
    title = models.CharField(max_length=100)
    # image = models.ImageField(upload_to='photos')
    image = ThumbnailImageFiled(upload_to='photos')
    caption = models.CharField(max_length=250, blank=True)

    class Meta:
        ordering = ['title']

    def __unicode__(self):
        return self.title

    @permalink
    def get_absolute_url(self):
        return 'photo_detail', None, &#123;'object_id': self.id&#125;

### admin.py
from django.contrib import admin
from models import Photo, Item


class PhotoInline(admin.TabularInline):  # admin.StackedInline
    model = Photo


class ItemAdmin(admin.ModelAdmin):
    inlines = [PhotoInline]

admin.site.register(Item, ItemAdmin)
admin.site.register(Photo)


## app/templates/app/
### base.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>&#123;% block title %&#125;&#123;% endblock %&#125;</title>
    &#123;% load static %&#125;
    <link rel="stylesheet" type="text/css" href="&#123;% static 'gallery/css/bootstrap.min.css' %&#125;">
</head>
<body>
&#123;% block header %&#125;
    <div class="container-fluid">
        <nav class="navbar navbar-default bg-primary navbar-static-top">
            <div class="navbar-header">
                <div class="navbar-brand">Gallery</div>
            </div>
            <ul class="nav navbar-nav nav-pills">
                <li><a href="&#123;% url 'index' %&#125;">Home</a></li>
                <li><a href="&#123;% url 'index' %&#125;">Gallery</a></li>
                <li><a href="#">Blog</a></li>
            </ul>
            <form class="navbar-form navbar-right" style="margin-right: 10px;">
                <div class="form-group">
                    <a type="button" class="btn btn-primary" href="admin/">Login</a>
                </div>
            </form>
        </nav>
    </div>
&#123;% endblock %&#125;
<div class="container-fluid">
    &#123;% block content %&#125;&#123;% endblock %&#125;
</div>
&#123;% block footer %&#125;
    <div class="container-fluid">
        <div class="navbar">
            <ul class="nav navbar-nav nav-pills">
                <li><a href="&#123;% url 'index' %&#125;">Home</a></li>
                <li><a href="&#123;% url 'index' %&#125;">Gallery</a></li>
                <li><a href="#">Blog</a></li>
            </ul>
        </div>
    </div>
    <script src="&#123;% static 'gallery/js/jquery.min.js' %&#125;"></script>
    <script src="&#123;% static 'gallery/js/bootstrap.min.js' %&#125;"></script>
&#123;% endblock %&#125;
</body>
</html>
### index.html
&#123;% extends 'gallery/base.html' %&#125;
&#123;% block title %&#125;Home&#123;% endblock %&#125;
&#123;% block content %&#125;
    <h1><a href="&#123;% url 'item_list' %&#125;">ItemList</a> </h1>
&#123;% endblock %&#125;

### items_list.html
&#123;% extends 'gallery/base.html' %&#125;
&#123;% block title %&#125;Home&#123;% endblock %&#125;
&#123;% block content %&#125;
    <div class="page-header"><h1>Showcase</h1></div>
    <div class="row">
        &#123;% for item in item_list|slice:":4" %&#125;
            <div class="col-sm-6 col-md-3">
                <div class="thumbnail">
                    &#123;% if item.photo_set.count %&#125;
                        <img src="&#123;&#123; item.photo_set.all.0.image.thumb_url &#125;&#125;" alt="...">
                    &#123;% else %&#125;
                        <span>No photos (yet)</span>
                    &#123;% endif %&#125;
                    <div class="caption">
                        <h3><a href="&#123;&#123; item.get_absolute_url &#125;&#125;">&#123;&#123; item.name &#125;&#125;</a></h3>
                        <p>&#123;&#123; item.description &#125;&#125;</p>
                        <p><a href="&#123;&#123; item.get_absolute_url &#125;&#125;" class="btn btn-primary">View Detail</a></p>
                    </div>
                </div>
            </div>
        &#123;% endfor %&#125;
    </div>
&#123;% endblock %&#125;
### items_detail.html
&#123;% extends 'gallery/base.html' %&#125;
&#123;% block title %&#125;Home&#123;% endblock %&#125;
&#123;% block content %&#125;
    <div class="page-header"><h1>&#123;&#123; item.name &#125;&#125;</h1></div>
    &#123;% for photo in item.photo_set.all %&#125;
        <div class="col-sm-6 col-md-3">
            <div class="thumbnail">
                <img src="&#123;&#123; photo.image.thumb_url &#125;&#125;" alt="...">
                <div class="caption">
                    <h3><a href="#">&#123;&#123; photo.title &#125;&#125;</a></h3>
                    <p>&#123;&#123; photo.caption &#125;&#125;</p>
                    <p><a href="&#123;% url 'photos_detail' photo.id %&#125;" type="button" class="btn btn-primary">Photo
                        Detail</a></p>
                </div>
            </div>
        </div>
    &#123;% endfor %&#125;
&#123;% endblock %&#125;

### photos_detail.html
&#123;% extends 'gallery/base.html' %&#125;
&#123;% block title %&#125;Home&#123;% endblock %&#125;
&#123;% block content %&#125;
    <div class="page-header"><h1>&#123;&#123; photo.title &#125;&#125;</h1></div>
    <h3><a href="#">&#123;&#123; photo.caption &#125;&#125;</a></h3>
    <img src="&#123;&#123; photo.image.url &#125;&#125;" alt="...">
    </div>
&#123;% endblock %&#125;
