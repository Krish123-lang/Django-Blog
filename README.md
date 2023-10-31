# Basic Blog App

1. `django-admin startproject mysite .`
2. `python3 manage.py startapp app`
3. `settings.py`
```
INSTALLED_APPS = [
    ...
    "app",
]

TEMPLATES = [
    {
        ...
        "DIRS": [BASE_DIR / 'templates'],
        ...
    },
]
```
4. `models.py`
```
from django.db import models
from django.contrib.auth.models import User

STATUS = (
    (0, "Draft"),
    (1, "Publish")
)
class Post(models.Model):
    title = models.CharField(max_length=200, unique=True)
    slug = models.SlugField(max_length=200, unique=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='blog_posts')
    updated_on = models.DateTimeField(auto_now=True)
    content = models.TextField()
    created_on = models.DateTimeField(auto_now_add=True)
    status = models.IntegerField(choices=STATUS, default=0)

    class Meta:
        ordering = ['-created_on']

    def __str__(self):
        return self.title
```
5. `admin.py`
```
from django.contrib import admin
from .models import Post

class PostAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug', 'status', 'created_on')
    list_filter = ("status",)
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}

admin.site.register(Post, PostAdmin)
```
6. `urls.py(blog)`
```
from . import views
from django.urls import path

urlpatterns = [
    path('', views.PostList.as_view(), name='home'),
    path('<slug:slug>/', views.PostDetail.as_view(), name='post_detail'),
]
```
7. `views.py`
```
from django.shortcuts import render
from django.views import generic
from .models import Post

class PostList(generic.ListView):
    queryset = Post.objects.filter(status=1).order_by('-created_on')
    template_name = 'index.html'

class PostDetail(generic.DetailView):
    model = Post
    template_name = 'post_detail.html'
```
8. `Create Templaets`
```
base.html
index.html
post_detail.html
sidebar.html
```
## Templates
1. `base.html`
```
{% block content %}
{% endblock content %}
```
2. `index.html`
```
{% extends "base.html" %}
{% block content %}

{% for post in post_list %}
    <h2 class="card-title">{{ post.title }}</h2>
    <p class="card-text text-muted h6">{{ post.author }} | {{ post.created_on}} </p>
    <p class="card-text">{{post.content|slice:":200" }}</p>
    <a href="{% url 'post_detail' post.slug  %}" class="btn btn-primary">Read More &rarr;</a>
{% endfor %}

{% block sidebar %} {% include 'sidebar.html' %} {% endblock sidebar %}

{% endblock content %}
```
3. `sidebar.html`
```
This is sidebar
```
4. `post_detail.html`
```
{% extends 'base.html' %} 
{% block content %}

<h1>{% block title %} {{ object.title }} {% endblock title %}</h1>
<p class=" text-muted">{{ post.author }} | {{ post.created_on }}</p>
<p class="card-text ">{{ object.content | safe }}</p>
{% block sidebar %} {% include 'sidebar.html' %} {% endblock sidebar %}

{% endblock content %}
```