---
title: "Crontab的使用"
date: 2017-08-30T01:37:56+08:00
lastmod: 2017-08-30T01:37:56+08:00
draft: false
tags: ["Linux", "Crontab"]
categories: ["服务器"]
author: "Liron"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---


#### 简介
crontab是linux中自带的计划任务工具

- 被周期性执行的任务我们称为Cron Job
- 周期性执行的任务列表我们称为Cron Table

#### 实践
- 检查cron服务
    - 检查crontab工具是否安装`crontab -l`
    - 检查crond服务是否启动`service crond status`
- 安装
    - `yum install vixie-cron`
    - `yum install crontabs`

#### 实例
- 添加一个简单的job用于测试，使用`crontab -e`命令并添加
```
*/1 * * * * date >> /tmp/cron11.log
```
一分钟以后检查`/tmp/cron11.log`文件中是否有写入，如果有表示cron运行正常

#### Crontab的组成
- `crond`系统服务
    - 每分钟都会从配置文件中刷新定时任务
    - 执行定时任务
- 配置文件
    - 文件方式设置定时任务
- 配置工具`crontab`
    - 用于调整定时任务

#### Crontab任务的格式

- 示例
    - 每晚21:30重启apache
    ```
    30 21 * * * service httpd restart
    ```
    - 每月1、10、22日的4：45重启apache
    ```
    45 4 1,10,22 * * service httpd restart
    ```
    - 每月1到10日的4:45重启apache
    ```
    45 4 1-10 * * service httpd restart
    ```
    - 每2分钟重启apache
    ```
    */2 * * * * service httpd restart
    ```
    - 晚上11点到早上7点之间，每隔1小时重启apache
    ```
    0 23-7/1 * * * service httpd restart
    ```
    - 每天18:00至23:00之间每隔30分钟重启apache
    ```
    0,30 18-23 * * * service httpd restart
    0-59/30 18-23 * * * service httpd restart
    ```
- 小结
    - `*`表示任何时候都匹配
    - 可用`A,B,C`表示`A`或者`B`或者`C`时执行命令
    - 可用`A-B`表示`A`到`B`之间执行命令
    - 可用`*/A`表示每`A`分钟（小时等）执行一次命令

#### crontab工具的使用
- 查看计划列表`crontab -l`
    - 查看指定用户的计划任务`crontab -l -u testuser`
- 编辑计划任务`crontab -e`
    - 编辑指定用户的计划任务`crontab -e -u testuser`

#### crontab的配置文件

```
[vagrant@localhost etc]$ cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```
#### crontab日志
`/var/log/cron`

#### 注意事项
- `%`需要加`\`转义
- 第三个和第五个域之间执行的是`或`操作
    - 四月的第一个星期日早晨1：59分重启apache
    ```
    59 1 1-7 4 0 service httpd restart
    ```
    上面的写法是容易犯的错误，这种写法实际的执行结果的4月的1到7日的1：59都重启apache。正确的写法应该是：
    ```
    59 1 1-7 4 * test `date +\%w` -eq 0 && service httpd restart
    ```
- 分钟数设置为`*`的错误，例如如果要设置一个每两小时都执行的任务，如果将分钟设置为`*`实际上匹配到的时候这个小时的每一分钟。所以分钟应该设置成`0`
