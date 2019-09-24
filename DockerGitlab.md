---
title: 基于Docker部署Gitlab教程
date: 2019-05-22 10:58:46
tags: [Docker,Gitlab,Help]
comment: true
toc: true
---

# 什么是 GitLab

GitLab 是利用 Ruby on Rails 一个开源的版本管理系统，实现一个自托管的 Git 项目仓库，可通过 Web 界面进行访问公开的或者私人项目。它拥有与 Github 类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序 (Wall) 进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。

<!--more-->

# 基于 Docker 安装 GitLab

首先进入到/usr/local/docker/gitlab目录下（若没有则创建），创建docker-compose.yml配置文件。

```bash
cd /usr/local/docker/gitlab
vi docker-compose.yml
```

使用 Docker 来安装和运行 GitLab 中文版,`docker-compose.yml` 配置如下：

```bash
version: '2'
services:
    gitlab:
      image: 'twang2218/gitlab-ce-zh:11.1.4'
      restart: always
      hostname: '192.168.41.131'  #你的linux虚拟机的IP地址
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://192.168.41.131'
          gitlab_rails['time_zone'] = 'Asia/Shanghai'
      ports:
        - '80:80'
        - '8443:443'
        - '2222:22'
      volumes:
        - /usr/local/docker/gitlab/config:/etc/gitlab
        - /usr/local/docker/gitlab/data:/var/opt/gitlab
        - /usr/local/docker/gitlab/logs:/var/log/gitlab
```

使用`docker-compose up`命令启动部署。

部署完成后，在浏览器输入`192.168.41.131`即可访问Gitlab。

# 关于配置多个SSH的问题

由于我本身还在使用Github，所以我的电脑上配置了一个Github SSH的Key，而现在我又搭建了一个本地托管服务器，并且希望通过SSH免密登陆，所以我需要配置第二个SSH，这里经常会出现冲突问题，有两种解决办法。

## 方法一

首先，生成第二个Gitlab使用的SSH公钥与私钥。

```
$ ssh-keygen -t rsa -C "2email@github.com” -f ~/.ssh/id_gitlab_rsa
```

此时，你的`.ssh`文件夹中应该有`id_gitlab_rsa``id_gitlab_rsa.pub``id_github_rsa``id_gitlab_hub.pub`四个文件，分别对应两个平台的公钥与私钥。

添加配置文件`config`如下：

```bash
# gitlab
Host 192.168.41.131 #Host可以看作是一个你要识别的模式，对识别的模式，进行配置对应的的主机名和ssh文件（可以直接填写ip地址）
    HostName 192.168.41.131  #本地Gitlab服务器IP
    Port 2222  #Gitlab端口号
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_gitlab #指明上面User对应的identityFile路径
    User GiottoLee #登录名（如gitlab的username）
# github
Host github
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_github
```

然后，添加私钥：

```bash
$ ssh-add ~/.ssh/id_rsa $ ssh-add ~/.ssh/github_rsa
```

如果执行ssh-add时提示”Could not open a connection to your authentication agent”，可以先执行命令:

```bash
$ ssh-agent bash
```

然后再重新运行ssh-add命令即可。

最后通过`SSH -T`命令进行测试：

```bash
$ ssh -T git@github.com
Hi GiottoLee! You've successfully authenticated, but GitHub does not provide shell access.
$ ssh -T git@192.168.41.131
Welcome to GitLab, @GiottoLee!
```

测试通过，现在两个SSH可同时使用了。

## 方法二

这个方法是我在搞清楚方法一之后才发现的，一句话就可以说清楚：

```
直接把github的公钥复制给Gitlab就好了，用一把钥匙访问两个仓库！Shit！
```

# 具体操作

Gitlab的具体操作与Github操作相同，此处不再赘述，如有需要可以翻看之前的那篇文章[Git版本控制及远程仓库的使用](http://giottolee.com/2019/04/08/GitHelpDoc/)]。