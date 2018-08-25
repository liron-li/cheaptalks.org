---
title: "使用 Travis CI 部署到自己的服务器"
date: 2018-08-25T17:53:58+08:00
draft: false
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
即可安装travis客户端。如果你在本地执行加密操作，你需要把秘钥文件下载到本地

```
cd /path/to/project
travis login
travis encrypt-file ~/.ssh/id_rsa --add
```

加密这里有个大坑，Windows不支持，以下引用自官方文档

> Caveat There is a report of this function not working on a local Windows machine. Please use a Linux or OS X  machine. 

#### 实例

```
language: go
go:
  - '1.10'
branches:
  only:
  - master
install:
  - wget https://github.com/gohugoio/hugo/releases/download/v0.47.1/hugo_0.47.1_Linux-64bit.tar.gz
  - tar -xzf hugo_0.47.1_Linux-64bit.tar.gz
  - mv hugo /usr/local/bin
script:
  - cd $TRAVIS_BUILD_DIR && hugo
addons:
  ssh_known_hosts:
  - $HOST
after_success:
  - chmod 600 ~/.ssh/id_rsa
  - ssh travis@$HOST -o StrictHostKeyChecking=no 'cd /var/www/cheaptalks.org && git pull && hugo'
notifications:
  email:
  - liron.li@outlook.com
env:
  global:
    secure: qNWNmTijIuBzv3faSIj6S6wjwGvAeV3CFyavN66NW6D7938QZtyLOMvttvwB034j+1bS5udkfZo3pLddu7HFEwlxnwXta5sj9IhxIl2Gxdu+F26D1tYE1hOcpqdS5lwTgJMyOYqpzpbELAC6W06gRcMB3d+mX0SxrF5orjuEvrDlX+hqBgpC+fG6qfQ09j18NHBLTkFKHqO3T7R4vP7uw8XMmhHoeU0vKbdw8NlRvh+iZgApfD0qsIPIYEKvsTOJHoSkluY8gVlkGcSa80M1iK94zJePVLHu8kqPANNLv5Y/mgS31DO48bfnjreHiMMQ26OCTKKMgmgExiozofLDkMgVuhxV6Zjec1cEiiScWQYKfEn+ka3nKX6tpdnTrOuCH3ha2AlERXQN0b2FduAsiuqtBkReoeEa3HY0dsSFtt8BBlKONrEtq9Ntp7g5TvyILN0/OcYkqTUQoytwLLudx1NhKwOjojy3CBy0Uinb0eS1Rpln9ZeR11e3TG5PzukUqYVFSXYWAZ9t92qdfmX6syUUC27+ki9CPLo5QrNsk55vBfei18QzFxoIr0faJ9n+MzMkVHDBdNjrvEF+zVIevee8uaBKM1g5p5CZEqEUyMiXyfh7P23Z5KPbzXBHPStrmZCenx4lKRhYbe9g6y64uH4Hu6s3HUGBr4F5g97SQZk=
before_install:
  - openssl aes-256-cbc -K $encrypted_37764402e94c_key -iv $encrypted_37764402e94c_iv
  -in id_rsa.enc -out ~/.ssh/id_rsa -d
```