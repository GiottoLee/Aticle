---
title: Git版本控制及远程仓库的使用
date: 2019-04-08 21:08:36
tags: [版本控制, 远程仓库,Help]
toc: true

---

## 配置Git

- ### git --version &emsp;查看Git版本
```
PS G:\DOC> git --version  
git version 2.21.0.windows.1
```

<br/>

- ### git comnfig --global user.name "GiottoLee" &emsp; 配置全局用户名

- ### git comnfig --global user.email "xxx@outlook.com" &emsp; 配置全局用户名

- ### git comfig --list &emsp; 查看当前所有项目

```
PS G:\DOC> git config --list  
    core.symlinks=false  
    core.autocrlf=true  
    core.fscache=true  
    color.diff=auto  
    color.status=auto  
    color.branch=auto  
    color.interactive=true  
    help.format=html  
    rebase.autosquash=true  
    http.sslbackend=openssl  
    http.sslcainfo=F:/Git/mingw64/ssl/certs/  ca-bundle.crt  
    credential.helper=manager  
    user.name=GiottoLee  
```

<br/>

## 创建仓库

- ### git init &emsp; 初始化仓库
```
PS G:\DOC\GitDoc> git init  
Reinitialized existing Git repository in G:/DOC/GitDoc/.git/
```

- ### git clone [ *连接* ] &emsp; 克隆仓库

<br/>

## 基本用法

- ### git status &emsp; 查看状态
```
PS G:\DOC\GitDoc> git status  
On branch master  
No commits yet  
Untracked files:  
  (use "git add <file>..." to include in what will be committed)  
        "Git\344\275\277\347\224\250\346\225\231\347\250\213.md"  
nothing added to commit but untracked files present (use "git add" to track)  
```

- ### git add . &emsp;将所有修改添加至暂存区
```
PS G:\DOC\GitDoc> git add .
```

- ### git add [ *File name* ] &emsp;将指定文件添加至暂存区
~~~
PS G:\DOC\GitDoc> git add HowToUseGit.md
~~~

- ### git commit -m " *备注* " &emsp; 提交版本
```
PS G:\DOC\GitDoc> git commit -m "Contribute how to use Git tools."
[master (root-commit) 3ccc1ff] Contribute how to use Git tools.
 1 file changed, 55 insertions(+)
 create mode 100644 "Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
```

- ### git log &emsp; 查看历史版本
```
PS G:\DOC\GitDoc> git log
commit 3ccc1ff91f3e2a6f9ed5908f414717893d9e3928 (HEAD -> master)
Author: GiottoLee <giottolee@outlook.com>
Date:   Sun Apr 7 21:17:44 2019 +0800

    Contribute how to use Git tools.
```

- ### git log -p &emsp; 查看历史版本附加详细信息
~~~
PS G:\DOC\GitDoc> git log -p
commit 3ccc1ff91f3e2a6f9ed5908f414717893d9e3928 (HEAD -> master)
Author: GiottoLee <giottolee@outlook.com>
Date:   Sun Apr 7 21:17:44 2019 +0800

    Contribute how to use Git tools.

diff --git "a/Git\344\275\277\347\224\250\346\225\231\347\250\213.md" "b/Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
new file mode 100644
index 0000000..dc41cd9
--- /dev/null
+++ "b/Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
@@ -0,0 +1,55 @@
+# Git使用教程
+
+## **配置Git**
+
+- ### git --version &emsp;查看Git版本
+> PS G:\DOC> git --version  
+git version 2.21.0.windows.1
+

...

+
+- ### git add . &emsp;将所有修改添加至暂存区
\ No newline at end of file
~~~


- ### git status &emsp; 查看当前状态
```
PS G:\DOC\GitDoc> git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   "Git\344\275\277\347\224\250\346\225\231\347\250\213.md"

no changes added to commit (use "git add" and/or "git commit -a")
```

- ### git checkout *版本号* &emsp; 穿越到指定的历史节点
```
PS G:\DOC\GitDoc> git checkout f921330
```

- ### git checkout - &emsp; 回到上一个版本节点
~~~
PS G:\DOC\GitDoc> git checkout -
~~~

<br/>

## 三种状态

1. Modified &emsp; 修改状态
2. Staged &emsp; 暂存状态
3. Committed &emsp; 提交状态

<br/>

## 标签Tag

- ### git tag &emsp; 查看标签
~~~
PS G:\DOC\GitDoc> git tag
NotFinished
~~~

- ### git tag -a [ *标签名* ] -m "*备注*" &emsp; 添加标签
~~~
PS G:\DOC\GitDoc> git tag -a NotFinished -m "New content added"
~~~

- ### git tag -a [ *标签名* ] -m "*备注*" [ *版本号* ] &emsp; 添加标签
~~~
PS G:\DOC\GitDoc> git tag -a NotFinished -m "New content added" f2165
~~~

- ### git show [ *标签名* ] &emsp; 查看某标签的详细信息
~~~bash
PS G:\DOC\GitDoc> git show NotFinished
tag NotFinished
Tagger: GiottoLee <giottolee@outlook.com>
Date:   Sun Apr 7 21:54:26 2019 +0800

New content added

commit 0c308273691037c07875a3e1239670e94d361722 (HEAD -> master, tag: NotFinished)
Author: GiottoLee <giottolee@outlook.com>
Date:   Sun Apr 7 21:53:10 2019 +0800

    New content added.

diff --git "a/Git\344\275\277\347\224\250\346\225\231\347\250\213.md" "b/Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
index dc41cd9..5f97e4b 100644
--- "a/Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
+++ "b/Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
@@ -3,8 +3,10 @@
~~~

 ## **配置Git**

 - ### git --version &emsp;查看Git版本

  ```bash
  PS G:\DOC> git --version  
  git version 2.21.0.windows.1
  ```

 - ### git comfig --list &emsp; 查看当前所有项目

~~~bash
PS G:\DOC> git config --list  
     core.symlinks=false
     core.autocrlf=true
     core.fscache=true
@@ -28,14 +31,17 @@ git version 2.21.0.windows.1
     http.sslcainfo=F:/Git/mingw64/ssl/certs/  ca-bundle.crt
     credential.helper=manager
     user.name=GiottoLee
~~~

 <br/>

 ## **创建仓库**

 - ### git init &emsp; 初始化仓库

  ```bash
  +PS G:\DOC\GitDoc> git init  
   Reinitialized existing Git repository in G:/DOC/GitDoc/.git/
  ```

  

 - ### git clone [ *连接* ] &emsp； 克隆仓库


## **基本用法**

 - ### git status &emsp; 查看状态


```bash
PS G:\DOC\GitDoc> git status  
 On branch master
 No commits yet
 Untracked files:
   (use "git add <file>..." to include in what will be committed)
         "Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
 nothing added to commit but untracked files present (use "git add" to track)
```


- ### git add . &emsp;将所有修改添加至暂存区

```
PS G:\DOC\GitDoc> git add .
```

- ### git add [*File name*] &emsp;将指定文件添加至暂存区
~~~
PS G:\DOC\GitDoc> git add HowToUseGit.md
~~~

- ### git commit -m "*备注*" &emsp; 提交版本
```
PS G:\DOC\GitDoc> git commit -m "Contribute how to use Git tools."
[master (root-commit) 3ccc1ff] Contribute how to use Git tools.
 1 file changed, 55 insertions(+)
 create mode 100644 "Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
```

- ### git log &emsp; 查看历史版本
```
PS G:\DOC\GitDoc> git log
commit 3ccc1ff91f3e2a6f9ed5908f414717893d9e3928 (HEAD -> master)
Author: GiottoLee <giottolee@outlook.com>
Date:   Sun Apr 7 21:17:44 2019 +0800

    Contribute how to use Git tools.
```

- ### git log -p &emsp; 查看历史版本附加详细信息
~~~
PS G:\DOC\GitDoc> git log -p
commit 3ccc1ff91f3e2a6f9ed5908f414717893d9e3928 (HEAD -> master)
Author: GiottoLee <giottolee@outlook.com>
Date:   Sun Apr 7 21:17:44 2019 +0800

    Contribute how to use Git tools.

diff --git "a/Git\344\275\277\347\224\250\346\225\231\347\250\213.md" b/Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
new file mode 100644
index 0000000..dc41cd9
--- /dev/null
+++ "b/Git\344\275\277\347\224\250\346\225\231\347\250\213.md"
@@ -0,0 +1,55 @@
+# Git使用教程
+
+## **配置Git**
+
+- ### git --version &emsp;查看Git版本
+> PS G:\DOC> git --version  
+git version 2.21.0.windows.1
+

...


- ### git add . &emsp;将所有修改添加至暂存区
\ No newline at end of file
~~~


- ### git status &emsp; 查看当前状态
```
PS G:\DOC\GitDoc> git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   "Git\344\275\277\347\224\250\346\225\231\347\250\213.md"

no changes added to commit (use "git add" and/or "git commit -a")
```

- ### git checkout *版本号* &emsp; 穿越到指定的历史节点
```
PS G:\DOC\GitDoc> git checkout f921330
```

- ### git checkout - &emsp; 回到上一个版本节点
~~~
PS G:\DOC\GitDoc> git checkout -
~~~

<br/>

## **三种状态**

1. Modified &emsp； 修改状态
2. Staged &emsp; 暂存状态
3. Committed &emsp; 提交状态

<br/>

## **标签Tag**

- ### git tag -a [*标签名*] -m "*备注*"

- ### git checkout [ *标签名* ] &emsp; 回退到该标签的版本

~~~
PS G:\DOC\GitDoc>git checkout NotFinished
~~~

<br/>

## 分支Branch

- ### git branch [ *分支名* ] &emsp; 创建分支
-
~~~
PS G:\DOC\GitDoc>git branch SubLine
Switched to branch 'SubLine'
~~~

- ### git checkout -b [ *分支名* ] &emsp; 创建并切换至分支

- ### git checkout master &emsp; 回到默认分支
~~~
PS G:\DOC\GitDoc>git checkout master
Switched to branch 'master'
~~~

- ### git log --all --graph &emsp; 图示全部历史纪录
~~~
PS G:\DOC\test> git log --all --graph
* commit 98e9c52b82abd46e83ed7c4d7433520b128e6edb (HEAD)
| Author: GiottoLee <giottolee@outlook.com>
| Date:   Sun Apr 7 22:44:57 2019 +0800
|
|     4
|
| * commit 0e5fb98813a701a105e8d59b56acd23abea93337 (master)
|/  Author: GiottoLee <giottolee@outlook.com>
|   Date:   Sun Apr 7 22:43:13 2019 +0800
|
|       3
|
* commit 9fcf68107bff21f89e61b06962ba93f3ec794129 (subline)
| Author: GiottoLee <giottolee@outlook.com>
| Date:   Sun Apr 7 22:43:03 2019 +0800
|
|     2
|
* commit bdcedd53fe633d870e8c2b708bf3703ae647f7de
  Author: GiottoLee <giottolee@outlook.com>
  Date:   Sun Apr 7 22:42:43 2019 +0800

      First commitPS 
~~~

<br/>

## 合并分支

- ### git merge [ *分支名* ] &emsp; 合并分支到当前分支
~~~
PS G:\DOC\GitDoc>git merge SubLine
~~~

~~~
1
2
//****************冲突部分********************
<<<<<<< HEAD 
3
=======
fixed
>>>>>>> subline
//********************************************
~~~

<br/>

## 远程仓库

- ### git remote add [ *远程名称* ] [ *远程地址* ] &emsp; 设置远程仓库
~~~
PS G:\DOC\GitDoc> git remote add GitHelpDoc git@github.com:G/GitHelpDoc.git
~~~

- ### git remote &emsp; 列出所有远程仓库
~~~
PS G:\DOC\GitDoc> git remote
GitHelpDoc
~~~

- ### git push -u [ *远程名称* ] [ *分支名* ] &emsp; 上传代码
~~~bash
PS G:\DOC\GitDoc> git push -u GitHelpDoc master
Warning: Permanently added the RSA host key for IP address '52.74.223.119' to the list of known hosts.
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 1.97 KiB | 404.00 KiB/s, done.
Total 6 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), done.
To github.com:GiottoLee/GitHelpDoc.git
 * [new branch]      master -> master
Branch 'master' set up to track remote branch 'master' from 'GitHelpDoc'.
PS G:\DOC\GitDoc>
~~~

- ### git clone [ *仓库地址* ] &emsp; 克隆远程仓库
~~~
PS G:\DOC\GitDoc> git clone git@github.com:G/GitHelpDoc.git
~~~

- ### git pull &emsp; 获取远程更新
~~~
PS G:\DOC\GitDoc> git pull
~~~

<br/>

## 多人远程合作

- ### git clone [ *仓库地址* ] [ *FileName* ] &emsp; 克隆远程仓库到指定文件夹
~~~
PS G:\DOC\GitDoc> git clone git@github.com:G/GitHelpDoc.git newFile
~~~

<br/>

[Github文档下载](https://github.com/GiottoLee/GitHelpDoc)
~~~
```

```
~~~