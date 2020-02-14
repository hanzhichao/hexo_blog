---
layout: post
title: Django开发LiveBlog
comments: true
toc: true
date: 2018-02-01 19:47:25
updated: 2018-02-01 19:47:25
categories:
tags: Django
---
Django 1.8.3

##项目级
###settings.py
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
)
### urls.py
from django.conf.urls import include, url
from django.contrib import admin

from liveupdate.views import *

urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^$', ObjectList.as_view()),
    url(r'^update-after/(?P<id>\d+)/$', update_after)
]
##App级
### models.py
from django.db import models


class Update(models.Model):
    timestamp = models.DateTimeField(auto_now_add=True)
    text = models.TextField()

    class Meta:
        ordering = ['-id']

    def __unicode__(self):
        return "[%s] %s" % (
            self.timestamp.strftime("%Y-%m-%d %H:%M:%S"),
            self.text
        )

### admin.py
from django.contrib import admin
from .models import Update

admin.site.register(Update)
### views.py
# -*- coding: utf-8 -*-
from django.http import HttpResponse
from django.core import serializers
from django.views.generic import ListView

from liveupdate.models import Update
from django.shortcuts import render

object_list = Update.objects.all()


class ObjectList(ListView):
    model = Update
    template_name = 'liveupdate/update_list.html'
    context_object_name = 'object_list'

    def get_context_data(self, **kwargs):
        item_list = super(ObjectList, self).get_context_data(**kwargs)
        return item_list


def update_after(request, id):
    response = HttpResponse()
    response['Content-Type'] = "text/javascript"
    response.write(serializers.serialize("json",
                                         Update.objects.filter(pk__gt=id)))
    return response

## App/Templates/App/
### update_list.html
<!DOCTYPE HTML>
<html>
<head>
    &#123;% load staticfiles %&#125;
    <title>Live Update</title>
    <style type="text/css">
        body &#123;
            margin: 30px;
            font-family: Arial;
            background: #fff;
        &#125;
        h1 &#123; background: #ccf; padding:20px; &#125;
        div.update &#123; width:100%; padding: 5px; &#125;
            div.even &#123; background: #ddd; &#125;
        div.timestamp &#123; float: left; font-weight: bold; &#125;
        div.text &#123; float: left; padding-left: 10px; &#125;
        div.clear &#123; clear: both; height: 1px; &#125;
    </style>
</head>
<body>
    <h1>Welcome to the Live Update!</h1>
    <p>This site will automatically refresh itself every minute with new content -
    Please <b>do not</b> reload the page!</p>
    &#123;% if not objects_list %&#125;
        <div id="update-holder">
            &#123;% for object in object_list %&#125;
            <div class="update &#123;% cycle even,odd %&#125;" id="&#123;&#123; object.id &#125;&#125;">
                <div class="timestamp">
                    &#123;&#123; object.timestamp|date:"Y-m-d H:i:s"&#125;&#125;
                </div>
                <div class="text">
                    &#123;&#123; object.text | linebreaksbr &#125;&#125;
                </div>
                <br/>
                <div class="clear"></div>
            </div>
            &#123;% endfor %&#125;
        </div>
    &#123;% else %&#125;
        <p>No updates yet - please check back later!</p>
    &#123;% endif %&#125;
    <script type="text/javascript" language="JavaScript" src="&#123;% static 'js/jquery.min.js' %&#125;"></script>
    <script type="text/javascript" language="JavaScript">
        function update()&#123;
            update_holder = $("#update-holder");
            most_recent = update_holder.find("div:first");
            $.getJSON("/update-after/" + most_recent.attr('id') + "/",
                function(data)&#123;
                    cycle_class = most_recent.hasClass("odd")
                        ? "even" :　"odd";
                    jQuery.each(data,function()&#123;
                        update_holder.prepend('<div id="' + this.pk
                            + '" class="update"' + cycle_class
                            + '"><div class="timestamp">'
                            + this.fields.timestamp
                            + '</div><div class="text">'
                            + this.fields.text
                            + '</div><div class="clear"></div></div>'
                        );
                        cycle_class = (cycle_class == "odd")
                            ? "even" : "odd";
                    &#125;);
                &#125;
            );
        &#125;
        $(document).ready(function()&#123;
            setInterval("update()",1000)
        &#125;)
    </script>
</body>
</html>
