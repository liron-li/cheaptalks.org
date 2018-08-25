---
title: "在Linux上部署Django"
date: 2017-08-30T01:37:56+08:00
lastmod: 2017-08-30T01:37:56+08:00
draft: false
tags: ["Linux", "Django"]
categories: ["服务器"]
author: "Liron"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

> 之前一直用`PHP`开发，从来没觉得部署是一件麻烦的事情，直到我遇到了`python`，
> 由于python2与python3的不兼容，而且大部分linux服务上自带的python版本都是2.x。
> 如果你使用python3开发那么部署python项目就有点麻烦了。 这里我记录下部署过程，以便以后忘记了可以有个参考，也是给需要的朋友提供一点微不足道的帮助。

#### 环境和工具
- ubuntu 14、ubuntu 16 (两个版本都部署过，基本没区别)
- Django 1.10
- virtualenv 15.1.0
- uWSGI 2.0.15
- nginx 1.10
- python3

#### 部署

##### 如何保持本地开发环境的依赖和线上环境一致？
 
众所周知`python`的依赖包是全局安装，和`php`的`composer`安装到项目的`vendor`目录有很大区别，也就是说依赖不和项目代码一起，所以python的项目需要建立`virtualenv`进行依赖隔离，避免不同项目依赖包的版本不一致引发的错误

###### 安装python3
一般来说linux上已经自带了python，在终端中输入python按`tab`即可查看服务器上现有的python版本

```
vagrant@vagrant-ubuntu-trusty:~$ python
python      python2     python2.7   python3     python3.4   python3.4m  python3m
```
发现并没有3.6 执行以下操作安装

```
sudo add-apt-repository ppa:fkrull/deadsnakes
sudo apt-get update

sudo apt-get install python3.6
sudo apt-get install python3.6-dev
```
安装完成后将`/usr/bin`中的python软连接执行对应的python版本即可。像这样：

```
ubuntu@ubuntu-xenial:/usr/bin$ ll python*
lrwxrwxrwx 1 root root       9 Dec 10  2015 python -> python2.7*
lrwxrwxrwx 1 root root       9 Dec 10  2015 python2 -> python2.7*
-rwxr-xr-x 1 root root 3546104 Nov 19 09:35 python2.7*
lrwxrwxrwx 1 root root      33 Nov 19 09:35 python2.7-config -> x86_64-linux-gnu-python2.7-config*
lrwxrwxrwx 1 root root      16 Dec 10  2015 python2-config -> python2.7-config*
lrwxrwxrwx 1 root root      11 Apr 25 08:11 python3 -> ./python3.6*
-rwxr-xr-x 2 root root 4460336 Nov 17 19:23 python3.5*
lrwxrwxrwx 1 root root      33 Nov 17 19:23 python3.5-config -> x86_64-linux-gnu-python3.5-config*
-rwxr-xr-x 2 root root 4460336 Nov 17 19:23 python3.5m*
lrwxrwxrwx 1 root root      34 Nov 17 19:23 python3.5m-config -> x86_64-linux-gnu-python3.5m-config*
-rwxr-xr-x 2 root root 4432640 Mar 22 10:18 python3.6*
lrwxrwxrwx 1 root root      33 Mar 22 10:18 python3.6-config -> x86_64-linux-gnu-python3.6-config*
-rwxr-xr-x 2 root root 4432640 Mar 22 10:18 python3.6m*
lrwxrwxrwx 1 root root      34 Mar 22 10:18 python3.6m-config -> x86_64-linux-gnu-python3.6m-config*
lrwxrwxrwx 1 root root      16 Apr 25 08:37 python3-config -> python3.6-config*
-rwxr-xr-x 1 root root     976 Nov 27  2015 python3-jsondiff*
-rwxr-xr-x 1 root root    3662 Nov 27  2015 python3-jsonpatch*
-rwxr-xr-x 1 root root    1342 Oct 24  2015 python3-jsonpointer*
lrwxrwxrwx 1 root root      10 Apr 25 08:36 python3m -> python3.6m*
lrwxrwxrwx 1 root root      17 Apr 25 08:37 python3m-config -> python3.6m-config*
lrwxrwxrwx 1 root root      16 Dec 10  2015 python-config -> python2.7-config*
```


###### virtualenv 的使用
安装python3后会自带pip3， 直接使用pip3安装`virtualenv`

```
pip3 install virtualenv
```
###### 创建一个新的python环境
第一步，创建目录
```
mkdir python-env
cd python-env
```
第二步，创建一个独立的Python运行环境，名为为django-env：
```
virtualenv --no-site-packages django-env --python=python3.6
```
第三步，切换到虚拟环境

```
source bin/activate
```

如果要退出虚拟环境使用

```
deactivate
```

传送门： [virtualenv手册](http://virtualenv-chinese-docs.readthedocs.io/en/latest/# "virtualenv手册") 

###### 安装依赖包
保持本地环境的依赖和线上一致，有几种方法，这里我使用最简单粗暴的方法：
第一步，导出本地环境的依赖，重定向到`txt`中

```
pip freeze > requirements.txt
```
第二步，安装依赖

```
pip install -r requirements.txt
```

至于怎样将代码上传到服务器，可以用`git`这个太简单不做说明

***注意：***

安装	`mysqlclient`包的时候可能会报错，根据错误提示，安装相关的包就可以了

##### uWSGI 是什么？
> uWSGI是一个Web服务器，它实现了WSGI协议、uwsgi、http等协议。Nginx中HttpUwsgiModule的作用是与uWSGI服务器进行交换。

###### uWSGI的安装
安装依旧是非常简单
```
pip3 install uwsgi
```
**注意：***
注意uwsgi最好退出虚拟环境再安装，并且安装和你python版本对应的uwsgi

###### uWSGI的配置
新建 uwsgi.ini 

```
[uwsgi]
virtualenv = /python-env/django1.10.env
uid = ubuntu
gid = ubuntu
socket = /tmp/mysite_uwsgi.sock
chdir = /python-env/django1.10.env/www/mysite
wsgi-file = /python-env/django1.10.env/www/mysite/mysite/wsgi.py
processes = 4
threads = 2
stats = 127.0.0.1:9191
daemonize = ./uwsgi.log
pidfile = ./uwsgi_pid.pid
#home = /python-env/django1.10.env/
#plugins=python36
```

启动：

```
uwsgi uwsgi.ini
```


###### uWSGI + Nginx
`Nginx`配置就不多说了

```
server {
    listen 80;
    #listen [::]:80 default_server ipv6only=on;

    # Make site accessible from http://localhost/
    server_name django.local.com;

    location / {
            include  uwsgi_params;
            uwsgi_pass unix:/tmp/mysite_uwsgi.sock;
    }
}
```

至此部署已经完成

***补充：***
安装完`python3`之后发现`pip3` 用不了了，报错如下：

```
Traceback (most recent call last):
  File "/var/www/mydir/virtualenvs/dev/bin/pip", line 5, in <module>
    from pkg_resources import load_entry_point
ImportError: No module named pkg_resources
```

百度谷歌查了好久终于解决了，方法如下：
```
wget https://bootstrap.pypa.io/ez_setup.py -O - | python
```

传送门： [stackoverflow](http://stackoverflow.com/questions/7446187/no-module-named-pkg-resources "stackoverflow") 