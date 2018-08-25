---
title: "Django学习笔记"
date: 2017-08-30T01:37:56+08:00
lastmod: 2017-08-30T01:37:56+08:00
draft: false
tags: ["Django"]
categories: ["Python"]
author: "Liron"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

#### 什么是Django
Django是一个基于python的高级web框架，它能够让开发人员进行高效且快点的开发，
高度集成（不需要自己造轮子） 并且开源免费

#### 起步
##### 创建项目
```
django-admin startproject myblog
```
##### 启动服务
进入项目目录
```
python manage.py runserver
```
在浏览器中输入`http://localhost:8000/` 即可看见`it  worked`页面

##### 目录结构
```
.
├── manage.py
└── myblog
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```
`wsgi.py` : WSGI (Python Web Server Gateway Interface) python服务器网关接口 python应用与web服务器之间的接口

`urls.py` : URL配置文件

`settings.py` : 配置文件

`__init__.py` : python中声明模块的文件

##### 创建应用
打开命令行，进入项目根目录，输入`python manage.py startapp blog` 然后添加应用名称到`settings.py`中的`INSTALLED_APPS`里

```
├── blog
│   ├── admin.py
│   ├── apps.py
│   ├── __init__.py
│   ├── migrations
│   │   ├── __init__.py
│   │   └── __pycache__
│   │       └── __init__.cpython-36.pyc
│   ├── models.py
│   ├── __pycache__
│   │   ├── admin.cpython-36.pyc
│   │   ├── __init__.cpython-36.pyc
│   │   └── models.cpython-36.pyc
│   ├── tests.py
│   └── views.py
├── db.sqlite3
├── manage.py
└── myblog
    ├── __init__.py
    ├── __pycache__
    │   ├── __init__.cpython-36.pyc
    │   ├── settings.cpython-36.pyc
    │   ├── urls.cpython-36.pyc
    │   └── wsgi.cpython-36.pyc
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

##### 配置url和视图
在`urls.py`中添加一个路由
```
from django.conf.urls import url
from django.contrib import admin

import blog.views as bv

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^index/', bv.index),
]
```

然后在`blog.views.py`中添加`index`方法
```
from django.shortcuts import render
from django.http import HttpResponse


def index(request):
    return HttpResponse('hello world')
```

##### 使用模板引擎
- 在app的目录下新建`templates`目录并在之下建立一个和app同名的目录
使用 render方法渲染模板
```
def index(request):
    return render(request, 'blog/index.html', dict(title="你好！世界"))
```
##### ORM 的使用

- 在应用的根目录下创建`models.py`，并引入models模块
- 创建类，继承`models.Model`， 该类即是一张数据表
- 使用前需要安装mysql驱动 `pip install mysqlclinet`
```
class Article(models.Model):
    title = models.CharField(max_length=32, default="Title")
    content = models.TextField(null=True)
```
执行 `python manage.py makemigrations`
`python manage.py migrate` 即可生成对应的表

获取一条记录并传递给模板
```
def index(request):
    article = models.Article.objects.get(pk=1)
    return render(request, 'blog/index.html', dict(article=article))
```
```
<h1>{{ article.title }}</h1>
<p>{{ article.content }}</p>
```

##### ADMIN的使用
- 创建admin用户
```
python manage.py createsuperuser
```
根据提示创建superuser



