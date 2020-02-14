---
layout: post
title: Django开发博客系统-版本三
comments: true
toc: true
date: 2018-02-01 19:33:13
updated: 2018-02-01 19:33:13
categories: 
tags: Django
---
根据Django of examples 1~3章开发 django==1.11.6

# settings.py
SITE_ID = 1

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
)

# SMTP
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_USE_TLS = False
EMAIL_HOST = 'smtp.sina.com'
EMAIL_PORT = 25
EMAIL_HOST_USER = 'test_results@sina.com'
EMAIL_HOST_PASSWORD = 'hanzhichao123'
DEFAULT_FROM_EMAIL = 'Test Results<test_results@sina.com>'
# urls.py
from django.conf.urls import url, include
from django.contrib import admin
from django.contrib.sitemaps.views import sitemap
from blog.sitemaps import PostSitemap
sitemaps = &#123;
    'posts': PostSitemap,
&#125;
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^blog/', include('blog.urls',
                           namespace='blog',
                           app_name='blog')),
    url(r'^sitemap\.xml$', sitemap, &#123;'sitemaps': sitemaps&#125;, name='django.contrib.sitemaps.views.sitemap'),
    url(r'^markdownx/', include('markdownx.urls')),
]
#blog/models.py
# -*- coding: utf-8 -*-
from __future__ import unicode_literals
from django.db import models
from django.urls import reverse
from django.utils import timezone
from django.contrib.auth.models import User
from taggit.managers import TaggableManager
class PublishedManager(models.Manager):
    def get_queryset(self):
        return super(PublishedManager, self).get_queryset().filter(status='published')
class Post(models.Model):
    STATUS_CHOICES = (
        ('draft', 'Draft'),
        ('published', 'Published'),
    )
    tags = TaggableManager()
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250, unique_for_date='publish')
    author = models.ForeignKey(User, related_name='blog_posts')
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='draft')
    objects = models.Manager()  # 自定义Manager
    published = PublishedManager()
    class Meta:
        ordering = ('-publish',)
    def __unicode__(self):
        return self.title
    def get_absolute_url(self):
        current_tz = timezone.get_current_timezone()
        db_local_time = current_tz.normalize(self.publish)
        return reverse('blog:post_detail', args=[
            db_local_time.year,
            db_local_time.strftime('%m'),
            db_local_time.strftime('%d'),
            self.slug
        ])
class Comment(models.Model):
    post = models.ForeignKey(Post, related_name='comments')
    name = models.CharField(max_length=80)
    email = models.EmailField()
    body = models.TextField()
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    active = models.BooleanField(default=True)
    class Meta:
        ordering = ('created',)
    def __unicode__(self):
        return 'Comment by &#123;&#125; on &#123;&#125;'.format(self.name, self.post)



#blog/admin.py



# -*- coding: utf-8 -*-
from __future__ import unicode_literals
from django.contrib import admin
from .models import Post, Comment
class PostAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug', 'author', 'publish', 'status')
    list_filter = ('status', 'created', 'publish', 'author')
    search_fields = ('title', 'body')
    prepopulated_fields = &#123;'slug': ('title',)&#125;
    raw_id_fields = ('author',)  # 显示外键的详细信息
    date_hierarchy = 'publish'  # 快捷搜索
    ordering = ['status', 'publish']  # 默认排序
class CommentAdmin(admin.ModelAdmin):
    list_display = ('name', 'email', 'post', 'created', 'active')
    list_filter = ('active', 'created', 'updated')
    search_fields = ('name', 'email', 'body')
admin.site.register(Post, PostAdmin)
admin.site.register(Comment, CommentAdmin)


#blog/forms.py


from django import forms
from .models import Comment
from markdownx.fields import MarkdownxFormField

class EmailPostForm(forms.Form):
    name = forms.CharField(max_length=25)
    email = forms.EmailField()
    to = forms.EmailField()
    comments = forms.CharField(required=False,
                               widget=forms.Textarea)
class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ('name', 'email', 'body')
class EditForm(forms.Form):
    myfield = MarkdownxFormField()



#blog/views.py

# -*- coding: utf-8 -*-
from __future__ import unicode_literals
from django.core.mail import send_mail
from django.shortcuts import render, get_object_or_404
from .models import Post
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from .forms import EmailPostForm, CommentForm, EditForm
from taggit.models import Tag
from django.db.models import Count
def post_list(request, tag_slug=None):
    object_list = Post.published.all()
    tag = None
    if tag_slug:
        tag = get_object_or_404(Tag, slug=tag_slug)
        object_list = object_list.filter(tags__in=[tag])
    paginator = Paginator(object_list, 3)
    page = request.GET.get('page')
    try:
        posts = paginator.page(page)
    except PageNotAnInteger:
        posts = paginator.page(1)
    except EmptyPage:
        posts = paginator.page(paginator.num_pages)
    return render(request, 'blog/post/list.html', &#123;
        'page': page,
        'posts': posts
    &#125;)
def post_detail(request, year, month, day, post):
    post = get_object_or_404(Post, slug=post,
                             status='published',
                             publish__year=year,
                             publish__month=month,
                             publish__day=day)
    comments = post.comments.filter(active=True)
    new_comment = None
    if request.method == 'POST':
        comment_form = CommentForm(data=request.POST)
        form = EditForm(data=request.POST)
        if comment_form.is_valid():
            new_comment = comment_form.save(commit=False)
            new_comment.post = post
            new_comment.save()
    else:
        form = EditForm()
        comment_form = CommentForm()
    # List of similar posts
    post_tags_ids = post.tags.values_list('id', flat=True)
    similar_posts = Post.published.filter(tags__in=post_tags_ids).exclude(id=post.id)
    similar_posts = similar_posts.annotate(same_tags=Count('tags')).order_by('-same_tags', '-publish')[:4]
    return render(request, 'blog/post/detail.html', &#123;
        'post': post,
        'comments': comments,
        'new_comment': new_comment,
        'comment_form': comment_form,
        'similar_posts': similar_posts,
        'form': form,
    &#125;)
def post_share(request, post_id):
    post = get_object_or_404(Post, id=post_id, status='published')
    cd = None
    if request.method == 'POST':
        form = EmailPostForm(request.POST)
        if form.is_valid():
            cd = form.cleaned_data
            post_url = request.build_absolute_uri(
                post.get_absolute_url()
            )
            subject = '&#123;&#125; (&#123;&#125;) recomends you reading "&#123;&#125;"'\
                .format(cd['name'], cd['email'], post.title)
            message = 'Read "&#123;&#125;" at &#123;&#125;\n\n&#123;&#125;\'s comments: &#123;&#125;'\
                .format(post.title, post_url, cd['name'], cd['comments'])
            send_mail(subject, message, 'test_results@sina.com', [cd['to']])
            sent = True
    else:
        form = EmailPostForm()
    return render(request, 'blog/post/share.html', &#123;
        'post': po
        'form': form,
    &#125;)

#blog/urls.py
from django.conf.urls import url, include
from . import views
from .feeds import LatestPostsFeed
urlpatterns = [
    url(r'^$', views.post_list, name='post_list'),
    url(r'^(?P<year>\d&#123;4&#125;)/(?P<month>\d&#123;2&#125;)/(?P<day>\d&#123;2&#125;)/(?P<post>[-\w]+)/$',
        views.post_detail, name='post_detail'),
    url(r'^(?P<post_id>\d+)/share/$', views.post_share, name='post_share'),
    url(r'^tag/(?P<tag_slug>[-\w]+)/$', views.post_list, name='post_list_by_tag'),
    url(r'feed/$', LatestPostsFeed(), name='post_feed'),
]

#blog/sitemaps.py
from django.contrib.sitemaps import Sitemap
from .models import Post
class PostSitemap(Sitemap):
    changefreq = 'weekly'
    priority = 0.9
    def items(self):
        return Post.published.all()
    def lastmode(self, obj):
        return obj.publish

#blog/feeds.py
from django.contrib.syndication.views import Feed
from django.template.defaultfilters import truncatewords
from .models import Post
class LatestPostsFeed(Feed):
    title = 'My blog'
    link = '/blog'
    description = 'New posts of my blog.'
    def items(self):
        return Post.published.all()[:5]
    def item_title(self, item):
        return item.title
    def item_description(self, item):
        return truncatewords(item.body, 30)

# blog/templatetags/blog_tags.py

from django import template
from django.db.models import Count
from django.utils.safestring import mark_safe
from ..models import Post
import markdown
from taggit.models import Tag
register = template.Library()
@register.simple_tag
def total_posts():
    return Post.published.count()
@register.inclusion_tag('blog/post/latest_posts.html')
def show_latest_posts(count=5):
    latest_posts = Post.published.order_by('-publish')[:count]
    return &#123;'latest_posts': latest_posts&#125;
@register.assignment_tag
def get_most_commented_posts(count=5):
    return Post.published.annotate(total_comments=Count('comments')).order_by('-total_comments')[:count]
@register.filter(name='markdown')
def markdown_format(text):
    return mark_safe(markdown.markdown(text))
@register.assignment_tag
def tag_cloud():
    return Tag.objects.all()

#blog/templates/blog/base.html
&#123;% load staticfiles %&#125;
&#123;% load blog_tags %&#125;
<!DOCTYPE html>
<html>
<head>
  <title>&#123;% block title %&#125;&#123;% endblock %&#125;</title>
  <link href="&#123;% static "css/bootstrap.min.css" %&#125;" rel="stylesheet">
</head>
<body>
<div class="container">
  <div class="col-md-9">
    &#123;% block content %&#125;
    &#123;% endblock %&#125;
  </div>
  <div class="col-md-3 bg-info">
    <h2><a href="&#123;% url 'blog:post_list' %&#125;">My blog</a></h2>
      <p>This is my blog. I've written &#123;% total_posts %&#125; posts so far.</p>
      <p><a href="&#123;% url 'blog:post_feed' %&#125;">Subscribe to my RSS feed</a></p>
      <h3>Latest posts</h3>
      &#123;% show_latest_posts 3 %&#125;
      <h3>Most commented posts</h3>
      &#123;% get_most_commented_posts as most_commented_posts %&#125;
      <ul>
          &#123;% for post in most_commented_posts %&#125;
              <li>
              <a href="&#123;&#123; post.get_absolute_url &#125;&#125;">&#123;&#123; post.title &#125;&#125;</a>
              </li>
          &#123;% endfor %&#125;
      </ul>
      <h3>All Tags</h3>
  &#123;% tag_cloud as tag_list %&#125;
  <ul>
      &#123;% for tag in tag_list %&#125;
      <li><a href="&#123;% url "blog:post_list_by_tag" tag.slug %&#125;">&#123;&#123; tag &#125;&#125;</a></li>
      &#123;% endfor %&#125;
  </ul>
  </div>
</div>
</body>
</html>
#blog/templates/blog/pagination.html
<span class="text-muted">
    &#123;% if page.has_previous %&#125;
        <a href="?page=&#123;&#123; page.previous_page_number &#125;&#125;">Previous</a>
    &#123;% endif %&#125;
    <span class="text-info">Page&#123;&#123; page.number &#125;&#125; of &#123;&#123; page.paginator.num_pages &#125;&#125;</span>
    &#123;% if page.has_next %&#125;
        <a href="?page=&#123;&#123; page.next_page_number &#125;&#125;">Next</a>
    &#123;% endif %&#125;
</span>
#blog/templates/blog/post/list.html
&#123;% extends "blog/base.html" %&#125;
&#123;% load blog_tags %&#125;
&#123;% block title %&#125;My Blog&#123;% endblock %&#125;
&#123;% block content %&#125;
    <h1>My Blog</h1><hr/>
    &#123;% if tag %&#125;
        <h2>Posts tagged with "&#123;&#123; tag.name &#125;&#125;"</h2>
    &#123;% endif %&#125;
  &#123;% for post in posts %&#125;
      <div class="panel panel-default"><div class="panel-body">
    <h2>
      <a href="&#123;&#123; post.get_absolute_url &#125;&#125;">
        &#123;&#123; post.title &#125;&#125;
      </a>
    </h2>
      <p>Tags:
      &#123;% for tag in post.tags.all %&#125;
          <a class="label label-default" href="&#123;% url "blog:post_list_by_tag" tag.slug %&#125;">&#123;&#123; tag.name &#125;&#125;</a>
          &#123;% if not forloop.last %&#125;&nbsp;&#123;% endif %&#125;
          &#123;% endfor %&#125;
          </p>
    <p class="date">
      Published &#123;&#123; post.publish &#125;&#125; by &#123;&#123; post.author &#125;&#125;
    </p>
    &#123;&#123; post.body|markdown|truncatechars_html:30 &#125;&#125;
      </div></div>
  &#123;% endfor %&#125;
  &#123;% include "blog/pagination.html" with page=posts %&#125;
<
&#123;% endblock %&#125;
#blog/templates/blog/post/detail.html
&#123;% extends "blog/base.html" %&#125;
&#123;% load blog_tags %&#125;
&#123;% block title %&#125;&#123;&#123; post.title &#125;&#125;&#123;% endblock %&#125;
&#123;% block content %&#125;
  <h1>&#123;&#123; post.title &#125;&#125;</h1>
  <p class="date">
    Published &#123;&#123; post.publish &#125;&#125; by &#123;&#123; post.author &#125;&#125;
  </p>
  &#123;&#123; post.body|markdown &#125;&#125;
    <p>
    <a href="&#123;% url 'blog:post_share' post.id %&#125;">Share this post</a>
    </p>
    <h2>Similar posts</h2>
    &#123;% for post in similar_posts %&#125;
        <p>
        <a href="&#123;&#123; post.get_absolute_url &#125;&#125;">&#123;&#123; post.title &#125;&#125;</a>
        </p>
        &#123;% empty %&#125;
        There are no similar posts yet.
    &#123;% endfor %&#125;
    <p>
    &#123;% with comments.count as total_comments %&#125;
        <h2>
        &#123;&#123; total_comments &#125;&#125; comment&#123;&#123; total_comments|pluralize &#125;&#125;
        </h2>
    &#123;% endwith %&#125;
    &#123;% for comment in comments %&#125;
        <div class="comment">
        <hr/>
        <p class="info">
            Comment &#123;&#123; forloop.counter &#125;&#125; by &#123;&#123; comment.name &#125;&#125;
            &#123;&#123; comment.created &#125;&#125;
        </p>
        &#123;&#123; comment.body|linebreaks &#125;&#125;
        </div>
        &#123;% empty %&#125;
        <p>There are no comments yet.</p>
    &#123;% endfor %&#125;
    &#123;% if new_comment %&#125;
        <h2>Your comment has been added.</h2>
    &#123;% else %&#125;
        <h2>Add a new comment</h2>
        <form action="." method="post">
        &#123;&#123; comment_form.as_p &#125;&#125;
        &#123;% csrf_token %&#125;
        <p><input type="submit" value="Add comment"></p>
        </form>
        <form method="POST" action="">&#123;% csrf_token %&#125;&#123;&#123; form.as_p &#125;&#125;</form>
&#123;&#123; form.media &#125;&#125;
    &#123;% endif %&#125;
&#123;% endblock %&#125;
#blog/templates/blog/post/latest_posts.html
<ul>
    &#123;% for post in latest_posts %&#125;
        <li>
        <a href="&#123;&#123; post.get_absolute_url &#125;&#125;">&#123;&#123; post.title &#125;&#125;</a>
        </li>
    &#123;% endfor %&#125;
</ul>
#blog/templates/blog/post/share.html
&#123;% extends "blog/base.html" %&#125;
&#123;% block title %&#125;Share a post&#123;% endblock %&#125;
&#123;% block content %&#125;
    &#123;% if sent %&#125;
        <h1>Email successfully sent</h1>
        <p>
        "&#123;&#123; post.title &#125;&#125;" was successfully sent to &#123;&#123; cd.to &#125;&#125;.
        </p>
    &#123;% else %&#125;
        <h1>Share "&#123;&#123; post.title &#125;&#125;" by e-mail</h1>
        <form action="." method="post">
        &#123;&#123; form.as_p &#125;&#125;
        &#123;% csrf_token %&#125;
        <input type="submit" value="Send e-mail">
        </form>
    &#123;% endif %&#125;
&#123;% endblock %&#125;
