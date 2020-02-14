---
layout: post
title: Framework-Vue开发mini_monitor
comments: true
toc: true
date: 2018-02-01 19:51:47
updated: 2018-02-01 19:51:47
categories:
tags: Django
---

根据麦子学院-Django开发监控系统开发
# requirements.txt
nose
djangorestframework
markdown
django-filter
# settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'monitor',
    'rest_framework',
]
#urls.py
from django.conf.urls import url, include
from django.contrib import admin
# from django.contrib.auth.models import User
# from rest_framework import routers, serializers, views
from monitor import views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^api-auth/',include('rest_framework.urls', namespace='rest_framework')),

    url(r'^api/apps/$',views.app_list),
    url(r'^api/apps/(?P<pk>[0-9]+)/$',views.app_detail),

    url(r'^api/app_statistics/$',views.app_statistics_list),
    url(r'^api/app_statistics/(?P<pk>[0-9]+)/$',views.app_statistics_detail),

    url(r'^api/app_history/$',views.app_history_list),
    url(r'^api/app_history/(?P<pk>[0-9]+)/$',views.app_history_detail),

    url(r'^api/groups/$',views.group_list),
    url(r'^api/groups/(?P<pk>[0-9]+)/$',views.group_detail),

    url(r'^api/hosts/$',views.host_list),
    url(r'^api/hosts/(?P<pk>[0-9]+)/$',views.host_detail)
]

#monitor/util/util.py
def get_ip(request):
    x_forwarded_for = request.META.get('HTTP_X_FORWARDED_FOR')
    if x_forwarded_for:
        ip = x_forwarded_for.split(',')[-1].strip()
    else:
        ip = request.META.get('REMOTE_ADDR')
    return ip
#monitor/models.py


from django.db import models
from datetime import datetime


# Create your models here.
class App(models.Model):
    class Meta:
        db_table = 'app'

    name = models.CharField(max_length=128)
    host_ip = models.IntegerField()
    group_id = models.IntegerField()
    configuration = models.TextField()
    status = models.CharField(max_length=12)
    message = models.TextField()
    enable = models.IntegerField()
    last_update = models.DateTimeField()
    

    @classmethod
    def create(cls, name, host_ip, status, message, enable, group_id,last_update):
        app = cls(name=name,
            host_ip=host_ip,
            status=status,
            message=message,
            enable=enable,
            group_id=group_id,
            last_update=last_update
            )
        return app

class AppStatistics(models.Model):
    class Meta:
        db_table = 'app_statistics'

    app_id = models.IntegerField()
    statistics = models.CharField(max_length=256)
    time = models.DateTimeField(auto_now=True)

    @classmethod
    def create(cls,app_id,statistics):
        appStatistics = cls(app_id=app_id,statistics=statistics,)
        return appStatistics    

class AppHistory(models.Model):
    class Meta:
        db_table = 'app_history'

    app_id = models.IntegerField()
    status = models.CharField(max_length=32)
    message = models.TextField(null=True)
    time = models.DateTimeField(auto_now=True)

    def convert_to_epoc(self):
        return self.time.strftime("%Y-%m-%d %H:%M:%S")

    @classmethod
    def create(cls, app_id, status, message):
        group = cls(app_id=app_id, status=status,message=message)
        return group

class Group(models.Model):
    class Meta:
        db_table = 'app_group'

    unique_name = models.CharField(max_length=32)
    display_name = models.CharField(max_length=32)

    @classmethod
    def create(cls, unique_name, display_name):
        group = cls(unique_name=unique_name, display_name=display_name)
        return group

class Host(models.Model):
    class Meta:
        db_table = 'app_host'

    name = models.CharField(max_length=32)
    ip = models.CharField(max_length=64)
    description = models.CharField(max_length=256, null=True)

    @classmethod
    def create(cls, ip, name=''):
        group = cls(ip=ip, name=name)
        return group
#serializers.py
from rest_framework import serializers
from monitor.models import App,AppStatistics,AppHistory,Group,Host
class AppSerializer(serializers.ModelSerializer):
    last_update = serializers.ReadOnlyField(source='convert_to_epoc')
    class Meta:
        model = App
        fields = ('id','name','host_ip','configuration','status',
            'message','enable','last_update')
class ManageAppSerializer(serializers.ModelSerializer):
    last_update = serializers.ReadOnlyField(source='convert_to_epoc')
    class Meta:
        model = App
        fields = ('id','name','host_ip','group_id','configuration','status',
            'message','enable','last_update')
class AppStatisticsSerializer(serializers.ModelSerializer):
    time = serializers.ReadOnlyField(source='convert_to_epoc')
    class Meta:
        model = AppStatistics
        fields = ('id','app_id','statistics','time')
class AppHistorySerializer(serializers.ModelSerializer):
    time = serializers.ReadOnlyField(source='convert_to_epoc')
    class Meta:
        model = AppHistory
        fields = ('app_id','status','message','time')
class GroupSerializer(serializers.ModelSerializer):
    class Meta:
        model = Group
        fields = ('unique_name', 'display_name', 'id')
class HostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Host
        fields = ('name', 'ip', 'description')

#monitor/views.py
# coding:utf-8
from django.shortcuts import render
from rest_framework.decorators import api_view
from monitor.serializers import AppSerializer, AppStatisticsSerializer
from monitor.serializers import AppHistorySerializer, GroupSerializer,HostSerializer
from monitor.models import App, AppStatistics, AppHistory, Group, Host
from rest_framework.response import Response
from django.views.decorators.csrf import csrf_exempt
from django.http.response import JsonResponse, HttpResponse
from util.util import get_ip
from datetime import datetime
import json


@api_view(['GET', 'POST'])
@csrf_exempt
def app_list(request):
    '''
    List all apps or create a new app
    '''
    if request.method == 'GET':
        group_id = request.GET.get('group_id',None)
        if group_id:
            tasks = App.objects.filter(enable=1).filter(group_id=group_id).all()
        else:
            tasks = App.objects.filter(enable=1).all()
        serializer = AppSerializer(tasks, many=True)
        return Response(serializer.data)
    elif request.method == 'POST':
        name = request.data.get('name')
        app = App.create(name,1,"OK","",1,2).save()
        serializer = AppSerializer(app, many=False)
        return JsonResponse(serializer.data,safe=False)

@api_view(['GET','PUT','DELETE'])
def app_detail(request, pk):
    '''
    Get,update or delete a specific app
    '''
    try:
        try:
            pk = int(pk)
            app = App.objects.get(pk=pk)
        except:
            app = App.objects.filter(name=pk).first()
    except App.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = AppSerializer(app)
        return JsonResponse(serializer.data)

    if request.method == 'DELETE':
        app.enable = 0
        return JsonResponse(serializer.data)

    if request.method == 'PUT':
        if not app:
            res = &#123;"code":405,"message":"Not found this app"&#125;
            return Response(data=res,status=405)

    ip = get_ip(request)
    if ip is not None:
        host = Host.objects.filter(ip=ip).first()
        if host is None:
            host = Host.create(ip)
            host.save()
    status = request.data.get("status")
    statistics = request.data.get("message",app.message)
    if status is None:
        res = &#123;"code":400,"message":"wong"&#125;
        return Response(data=res, status=400)
    app.status = status
    app.last_update = datetime.now()
    app.host_id = host.id
    app.save()
    if statistics:
        try:
            json.loads(statistics)
        except:
            res = &#123;"code":400,"message":"Statistics format must json"&#125;
            return Response(data=res,status=400)
        appStatistics = AppStatistics.create(statistics,app.id)
        appStatistics.save()
        serializer = AppSerializer(app)
    return JsonResponse(serializer.data)

def manage_detail(request, pk):
    try:
        pk = int(pk)
        app = App.objects.get(pk=pk)
    except:
        app = App.objects.filter(name=pk).first()
    if not app:
        return HttpResponse(status=404)

    elif request.method == 'GET':
        serializer = manageAppSerializer(app)
        return JsonResponse(serializer.data)

    elif request.method == 'POST':
        app.name = reqqest.data.get("name",app_name)
        app.host_id = request.data.get("host_id", app.host_id)
        app.group_id = request.data.get("group_id", app.group_id)
        app.configuration = request.data.get("configuration", app.configuration)
        app.save()
        serializer = manager_app_serializer(app, many=False)
        return JsonResponse(serializer.data, safe=False)


@api_view(['GET'])
@csrf_exempt
def app_statistics_list(request,pk):
    '''
    List all app_statisticss
    '''
    if request.method == 'GET':
        limit = int(request.GET.get('limit',12))
        start_date = request.GET.get('start_date',None)
        end_date = request.GET.get('end_date',None)
        if start_date and end_date:
            app_statistics_list = AppStatistics.objects.filter(app_id=pk).filter(time__range=(start_date,end_date)).order_by('-id').all()
        else:
            app_statistics_list = AppStatistics.objects.filter(app_id=pk).order_by('-id')[:limit].all()

        serializer = AppStatisticsSerializer(app_statistics_list, many=False)
        return JsonResponse(serializer.data, safe=False)


@api_view(['GET', 'POST'])
@csrf_exempt
def app_history_list(request):
    '''
    List all app_historys or create a new app_history
    '''
    if request.method == 'GET':
        limit = reqqest.GET.get('limit', 12)
        app_id = reqqest.GET.get('app_id')
        try:
            limit = int(limit)
            app_id = int(app_id)
        except:
            res = &#123;"code":404,"message":"Limit must be int"&#125;
            return Response(data=res,status=400)
        app_history_list = AppHistory.objects.filter(app_id=app_id).order_by('-id')[:limit]
        serializer = AppHistorySerializer(app_history_list, many=True)
        return Response(serializer.data)
    elif request.method == 'POST':
        app_id = request.data.get('app_id')
        status = request.data.get('status')
        message = request.data.get('message')
        if app_id and status:
            checkapp_history = AppHistory.objects.filter(app_id=app_id).first()
            if checkapp_history:
                res = &#123;"code":400,
                "message": "Ops! app history app_id already exists"&#125;
                return Response(data=res,status=400)
            
            checkip = AppHistory.objects.filter(status=status).first()
            if checkip:
                res = &#123;"code":400,
                "message": "Ops! app history status already exists"&#125;
                return Response(data=res,status=400)
        else:
            res = &#123;"code":400,
            "message":"Ops!app history app_id and status can't be null"

            &#125;
            return Response(data=res,status=400)
        app_history = AppHistory.create(app_id, status, message)
        app_history.save()
        serializer = AppHistorySerializer(app_history, many=False)
        return JsonResponse(serializer.data, safe=False)


@api_view(['GET', 'POST'])
@csrf_exempt
def group_list(request):
    '''
    List all groups or create a new group
    '''
    if request.method == 'GET':
        tasks = Group.objects.all()
        serializer = GroupSerializer(tasks, many=True)
        return Response(serializer.data)
    elif request.method == 'POST':
        # serializer = GroupSerializer(data=request.DATA)
        # if serializer.is_valid():
        #     serializer.save()
        #     return Response(serializer.data, status=201)
        unique_name = request.data.get('unique_name')
        status = request.data.get('status')
        message = request.data.get('message')
        if unique_name and status:
            checkgroup = Group.objects.filter(unique_name=unique_name).first()
            if checkgroup:
                res = &#123;"code":400,
                "message": "Ops! Unique name already exists"&#125;
                return Response(data=res,status=400)
        else:
            res = &#123;"code":400,
            "message":"Ops!Unique name and display name can't be null"

            &#125;
            return Response(data=res,status=400)
        group = Group.create(unique_name, display_name)
        group.save()
        serializer = GroupSerializer(group, many=False)
        return JsonResponse(serializer.data, safe=False)

@api_view(['GET','PUT','DELETE'])
def group_detail(request, pk):
    '''
    Get,update or delete a specific group
    '''
    try:
        group = Group.objects.get(pk=pk)
    except Group.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = GroupSerializer(group)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        group.unique_name = request.data.get('unique_name',group.unique_name)
        group.display_name = request.data.get('display_name',
            group.display_name)
        group.save()
        serializer = GroupSerializer(group)
        return JsonResponse(serializer.data)

    elif request.method == 'DELETE':
        group.delete()
        res = &#123;"code":200,
        "message":"Delete Suessus!"&#125;
        return Response(data=res,status=200)


@api_view(['GET'])
@csrf_exempt
def host_list(request):
    '''
    List all hosts
    '''
    if request.method == 'GET':
        tasks = Host.objects.all()
        serializer = HostSerializer(tasks, many=True)
        return Response(serializer.data)

@api_view(['GET','PUT','DELETE'])
def host_detail(request, pk):
    '''
    Get,update or delete a specific host
    '''
    try:
        host = Host.objects.get(pk=pk)
    except Host.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = HostSerializer(host)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        host.name = request.data.get('name',host.name)
        host.ip = request.data.get('ip',host.ip)
        host.save()
        serializer = HostSerializer(host)
        return JsonResponse(serializer.data)
# 前端----待完成
