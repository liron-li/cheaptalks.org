---
title: "记一次MySQL不能启动的奇怪问题"
date: 2017-08-30T01:37:56+08:00
lastmod: 2017-08-30T01:37:56+08:00
draft: false
tags: ["Mysql"]
categories: ["服务器"]
author: "Liron"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

今天升级了Ubuntu14机器上的mysql为5.7，升级完后发现不能启动mysql。在错误日志中看见这样一个错误
```
Could not create unix socket lock file /var/run/mysqld/mysqld.sock.lock.
```
初步判断是目录权限问题，但是查看这个目录是属于mysql用户的并且有读写权限。这就有点奇怪了，最终在`stackoverflow`上找到了答案

>Note to future travelers: It depends on your specific configuration but this is very likely an issue with apparmor. If you don't want to disable locking take a look at syslog and see if you're getting apparmor denies on that file.

>You'll see something like: apparmor="DENIED" operation="open" parent=29871 profile="/usr/sbin/mysqld" name="/run/mysqld/mysqld.sock.lock"

>And can fix it by adding /run/mysqld/mysqld.sock.lock rw to /etc/apparmor.d/usr.sbin.mysqld near the other /run/* entries and reloading apparmor.

按照这个步骤设置后重启`apparmor`即可

终于mysql可以启动了，但是又有了新的问题：
```
Table 'performance_schema.session_variables' doesn't exist
```
这个是由于升级mysql后配置信息没有更新引起的，执行下面的命令即可
```
mysql_upgrade -u root -p --force
```
