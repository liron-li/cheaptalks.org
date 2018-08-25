---
title: "使用webhook 自动化部署"
date: 2017-08-30T01:37:56+08:00
lastmod: 2017-08-30T01:37:56+08:00
draft: false
tags: ["git", "webhook"]
categories: ["服务器"]
author: "Liron"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

使用git提供的webhook功能可以轻松的实现自动化部署

### 使用步骤
创建RSA秘钥,因为后面要使用http服务器来执行脚本，所以这里给nginx的www-data用户创建秘钥。如果你使用的apache或其他服务器，换成对应的用户即可。
```
sudo -u www-data ssh-keygen -t rsa -C "xxx@xx.com"
```

创建成功后使用`cat`命令查看公钥内容,并将公钥添加到git服务中
```
root@iZ28:/var/www/# cat /var/www/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDnkRE95doDzC90j69xQZ+slylPlgdYm77mCjKaIOwyhFdTubbaPBiGo+1ztz+fm6v64ywgfMsb...

```

测试是否能访问git服务，这里使用的oschina的git
```
sudo -u www-data ssh -T git@git.oschina.net
```
返回`Welcome to Git@OSC, Anonymous!` 表示配置成功

测试www-data 是否有执行`git pull`的权限
```
sudo -u www-data git pull
```
如果提示没有权限则进行如下修改,给www-data用户git的权限
```
vim /etc/sudoers

```

```
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL
git ALL=(www-data) ALL


# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoer
```

编写脚本文件执行shell
```
<?php

$www_folder = '/var/www/blog/';

//执行指令
echo shell_exec(" cd $www_folder && git pull git@git.oschina.net:LeeRock/blog.git");
```

最后设置一个`url`访问这个脚本，并在git服务中配置webhook地址，配置完成后`git push`提交后会自动访问到指定的webhook地址就能实现自动更新了