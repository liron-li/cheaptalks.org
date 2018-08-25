---
title: "使用 Travis CI 部署到自己的服务器"
date: 2018-08-25T17:53:58+08:00
draft: true
tags: ["持续集成"]
categories: ["服务器"]
author: "Liron"

contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'

---

> Travis CI 是广泛使用的持续集成工具，怎么使用这里不做赘述，需要的小伙伴可以自行查询官方文档。本文章仅对部署到自己服务器做一个简单的说明

### Travis CI使用SSH免密登陆服务器

#### 登录服务器创建`travis`指定所有权限，并设置秘钥登录

```
	#新建用户
	useradd travis
	#修改密码（应该不是必要，但是万一以后需要用密码登陆呢）,按照提示设置密码。
	passwd travis
	#为用户添加添加权限
	vim /etc/sudoers
```

找到#Allow root to run any commands anywhere这一段注释，在下面新增一行（这里是指定所有权限，需谨慎）：
```
travis  ALL=(ALL)   ALL
```

#### 生成密钥对

```
root@hongkong:~$ su travis && cd ~
travis@hongkong:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/travis/.ssh/id_rsa): 
Created directory '/home/travis/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/travis/.ssh/id_rsa.
Your public key has been saved in /home/travis/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Sp5***********************NLb0us travis@hongkong
The key's randomart image is:
+---[RSA 2048]----+
|        oo+o.    |
|       o oo..    |
|      . o .o     |
|       =  .   .  |
|      . S. . + . |
|     o +  . * = o|
|      *    . B.*o|
|     o      =+Bo+|
|            +X%Eo|
+----[SHA256]-----+
travis@hongkong:~$ cd .ssh/
travis@hongkong:~/.ssh$ ls
id_rsa	id_rsa.pub

```

#### 将生成的公钥添加为受信列表

```
travis@hongkong:~/.ssh$ cat id_rsa.pub >> authorized_keys
```

#### 测试连接

```
travis@hongkong:~/.ssh$ ssh travis@00.00.00.00
The authenticity of host '00.00.00.00 (00.00.00.00)' can't be established.
ECDSA key fingerprint is SHA256:Ax4uI*********************Mchx4ZMw.
Are you sure you want to continue connecting (yes/no)? yes
```

连接后会在`known_hosts`中加入一条记录，这样下次连接时就不用输入`yes`了

#### 使用Travis客户端工具加密秘钥

工具依赖`ruby`，需要安装`ruby`环境，安装完成后执行：

```
gem install travis
```
即可安装travis客户端，