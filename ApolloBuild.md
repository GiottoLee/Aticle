---
title: 搭建携程Apollo分布式配置中心
date: 2019-06-01 21:02:31
tags: [Apollo,分布式,Help]
toc: true
comment: true
---

# 概述

随着Spring Cloud项目的越来越繁琐，配置文件越来越多，之前的本地存储方式就显得有些不够用了，一方面，本地存储的方式不够灵活，另一方面，如果需要更改配置，会十分繁琐并且需要重启服务，这对于一个系统来说十分不友好。

之前我也使用Docker+Gitlab的方式搭建了一个分布式配置中心，主要有三个问题：

1. 更改配置后需要通过gti手动上传；
2. 配置更改后服务需要重启才能获取到最新配置；
3. 需要通过命令更改环境配置。

之后，根据老师的建议，决定使用携程研发的开源配置管理中心—Apollo。

Apollo开发者提供了详尽的[部署文档]([https://github.com/ctripcorp/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E4%BB%8B%E7%BB%8D](https://github.com/ctripcorp/apollo/wiki/Apollo配置中心介绍))，但是对于小白显得不那么友好，所以我想从一个初学者的角度讲述我的Apollo部署过程。

<!--more-->

# Apollo特点

正是基于配置的特殊性，所以Apollo从设计之初就立志于成为一个有治理能力的配置发布平台，目前提供了以下的特性：

- **统一管理不同环境、不同集群的配置**
	- Apollo提供了一个统一界面集中式管理不同环境（environment）、不同集群（cluster）、不同命名空间（namespace）的配置。
	- 同一份代码部署在不同的集群，可以有不同的配置，比如zookeeper的地址等
	- 通过命名空间（namespace）可以很方便地支持多个不同应用共享同一份配置，同时还允许应用对共享的配置进行覆盖
- **配置修改实时生效（热发布）**
	- 用户在Apollo修改完配置并发布后，客户端能实时（1秒）接收到最新的配置，并通知到应用程序
- **版本发布管理**
	- 所有的配置发布都有版本概念，从而可以方便地支持配置的回滚
- **灰度发布**
	- 支持配置的灰度发布，比如点了发布后，只对部分应用实例生效，等观察一段时间没问题后再推给所有应用实例
- **权限管理、发布审核、操作审计**
	- 应用和配置的管理都有完善的权限管理机制，对配置的管理还分为了编辑和发布两个环节，从而减少人为的错误。
	- 所有的操作都有审计日志，可以方便地追踪问题
- **客户端配置信息监控**
	- 可以在界面上方便地看到配置在被哪些实例使用
- **提供Java和.Net原生客户端**
	- 提供了Java和.Net的原生客户端，方便应用集成
	- 支持Spring Placeholder, Annotation和Spring Boot的ConfigurationProperties，方便应用使用（需要Spring 3.1.1+）
	- 同时提供了Http接口，非Java和.Net应用也可以方便地使用
- **提供开放平台API**
	- Apollo自身提供了比较完善的统一配置管理界面，支持多环境、多数据中心配置管理、权限、流程治理等特性。不过Apollo出于通用性考虑，不会对配置的修改做过多限制，只要符合基本的格式就能保存，不会针对不同的配置值进行针对性的校验，如数据库用户名、密码，Redis服务地址等
	- 对于这类应用配置，Apollo支持应用方通过开放平台API在Apollo进行配置的修改和发布，并且具备完善的授权和权限控制
- **部署简单**
	- 配置中心作为基础服务，可用性要求非常高，这就要求Apollo对外部依赖尽可能地少
	- 目前唯一的外部依赖是MySQL，所以部署非常简单，只要安装好Java和MySQL就可以让Apollo跑起来
	- Apollo还提供了打包脚本，一键就可以生成所有需要的安装包，并且支持自定义运行时参数

# 分布式部署

根据部署文档所述，Quick Start部署方式虽然简单快捷，但是不建议使用于生产环境中。

> 为了让大家更快的上手了解Apollo配置中心，我们这里准备了一个Quick Start，能够在几分钟内在本地环境部署、启动Apollo配置中心。
>
> 考虑到Docker的便捷性，我们还提供了Quick Start的Docker版本，如果你对Docker比较熟悉的话，可以参考[Apollo Quick Start Docker部署](https://github.com/ctripcorp/apollo/wiki/Apollo-Quick-Start-Docker部署)通过Docker快速部署Apollo。
>
> 不过这里需要注意的是，Quick Start只针对本地测试使用，如果要部署到生产环境，还请另行参考[分布式部署指南](https://github.com/ctripcorp/apollo/wiki/分布式部署指南)。

所以综合考量，我选择使用分布式部署方式。

# 部署指南

## 部署环境

### OS

> 服务端基于Spring Boot，启动脚本理论上支持所有Linux发行版，建议[CentOS 7](https://www.centos.org/)。

操作系统我选用的是**Ubuntu 16.04**版本。

### Java

> - Apollo服务端：1.8+
> - Apollo客户端：1.7+
>
> 由于需要同时运行服务端和客户端，所以建议安装Java 1.8+。
>
> > 对于Apollo客户端，运行时环境只需要1.7+即可。
>
> > 注：对于Apollo客户端，如果有需要的话，可以做少量代码修改来降级到Java 1.6，详细信息可以参考[Issue 483](https://github.com/ctripcorp/apollo/issues/483)

Java的安装与环境变量配置在CSDN、百度上都有很多，只要按照步骤跟着做，不会有问题。安装完成后可以通过以下命令验证：

```
cgteam@cgteam:~/lizhe$ java -version
openjdk version "1.8.0_181"
OpenJDK Runtime Environment (build 1.8.0_181-8u181-b13-1ubuntu0.16.04.1-b13)
OpenJDK 64-Bit Server VM (build 25.181-b13, mixed mode)
```



这里我使用的是JDK1.8。

### MySQL

> - 版本要求：5.6.5+
>
> Apollo的表结构对`timestamp`使用了多个default声明，所以需要5.6.5以上版本。

MySQL同样根据CSDN教程安装即可，连接上MySQL后，可以通过如下命令检查：

```mysql
SHOW VARIABLES WHERE Variable_name = 'version';
```

| Variable_name | Value  |
| ------------- | ------ |
| version       | 5.7.11 |

## 环境

Apollo目前支持以下环境：

- DEV
	- 开发环境
- FAT
	- 测试环境，相当于alpha环境(功能测试)
- UAT
	- 集成环境，相当于beta环境（回归测试）
- PRO
	- 生产环境

具体可以查看[开发文档](https://github.com/ctripcorp/apollo/wiki/%E5%88%86%E5%B8%83%E5%BC%8F%E9%83%A8%E7%BD%B2%E6%8C%87%E5%8D%97#13%E7%8E%AF%E5%A2%83)。

## 部署步骤

根据开发者发布的文档来看，当我们在服务器部署Apollo时，需要将安装包在服务器端解压执行。开发者提供了两种获取安装包的方式：

1. 直接下载官方提供的安装包；
2. 通过源码自行构建。

这里我建议使用第二种，因为这样比较简单，不需要解压后挨个改配置，只要在打包前修改一次脚本文件即可。

通过Git或者下载Zip的方式获取[源码](<https://github.com/ctripcorp/apollo>)，源码如下。

![源码](/ApolloBuild/1559440284688.png)

### 创建数据库

部署Apollo需要使用到两个数据库：`ApolloConfigDB`和`ApolloPortalDB`。这一步十分简单，打开你的MySQL客户端，创建两个新的数据库，然后运行官方提供的sql文件即可。sql文件分别在：`\apollo-master\scripts\db\migration\configdb`以及`\apollo-master\scripts\db\migration\portaldb`目录下。

创建数据库：

![创建数据库](/ApolloBuild/1559440617638.png)

运行sql文件：

![运行sql文件-1](/ApolloBuild/1559441168549.png)

![运行sql文件-2](/ApolloBuild/1559440988926.png)

#### 验证

导入成功后，可以通过执行以下sql语句来验证：

**ApolloPortalDB:**

```mysql
select `Id`, `Key`, `Value`, `Comment` from `ApolloPortalDB`.`ServerConfig` limit 1;
```

| Id   | Key                | Value | Comment          |
| ---- | ------------------ | ----- | ---------------- |
| 1    | apollo.portal.envs | dev   | 可支持的环境列表 |

**ApolloConfigDB:**

```mysql
select `Id`, `Key`, `Value`, `Comment` from `ApolloConfigDB`.`ServerConfig` limit 1;
```

| Id   | Key                | Value                           | Comment       |
| ---- | ------------------ | ------------------------------- | ------------- |
| 1    | eureka.service.url | <http://127.0.0.1:8080/eureka/> | Eureka服务Url |

> 注意：ApolloPortalDB只需要在生产环境部署一个即可，而ApolloConfigDB需要在每个环境部署一套，如fat、uat和pro分别部署3套ApolloConfigDB。

#### 配置数据库

Apollo自身的一些配置存储在数据库里，所以我们需要对数据库进行一些简单的配置。

##### ApolloConfigDb

![ApolloConfigDB配置](/ApolloBuild/1559441787747.png)

**eureka.service.url** :Apollo需要将`Apollo-adminservice`以及`Apollo-portal`注册到Eureka上，由于`Apollo-configservice`本身就是一台Eureka服务器，所以这里只需填写配置`Apollo-configservice`所在服务的IP地址以及端口即可，如`http://192.168.13.13:8080/eureka/`。

*这里不建议使用`localhost`写法，后期运行`apollo-portal`有可能会报错。*

其他选项请根据自己需要自行配置。

> ##### 1. eureka.service.url - Eureka服务Url
>
> 不管是apollo-configservice还是apollo-adminservice都需要向eureka服务注册，所以需要配置eureka服务地址。 按照目前的实现，apollo-configservice本身就是一个eureka服务，所以只需要填入apollo-configservice的地址即可，如有多个，用逗号分隔（注意不要忘了/eureka/后缀）。
>
> 需要注意的是每个环境只填入自己环境的eureka服务地址，比如FAT的apollo-configservice是1.1.1.1:8080和2.2.2.2:8080，UAT的apollo-configservice是3.3.3.3:8080和4.4.4.4:8080，PRO的apollo-configservice是5.5.5.5:8080和6.6.6.6:8080，那么：
>
> 1. 在FAT环境的ApolloConfigDB.ServerConfig表中设置eureka.service.url为：
>
> ```
> http://1.1.1.1:8080/eureka/,http://2.2.2.2:8080/eureka/
> ```
>
> 1. 在UAT环境的ApolloConfigDB.ServerConfig表中设置eureka.service.url为：
>
> ```
> http://3.3.3.3:8080/eureka/,http://4.4.4.4:8080/eureka/
> ```
>
> 1. 在PRO环境的ApolloConfigDB.ServerConfig表中设置eureka.service.url为：
>
> ```
> http://5.5.5.5:8080/eureka/,http://6.6.6.6:8080/eureka/
> ```
>
> > 注1：这里需要填写本环境中全部的eureka服务地址，因为eureka需要互相复制注册信息
>
> > 注2：如果希望将Config Service和Admin Service注册到公司统一的Eureka上，可以参考[部署&开发遇到的常见问题 - 将Config Service和Admin Service注册到单独的Eureka Server上](https://github.com/ctripcorp/apollo/wiki/部署&开发遇到的常见问题#8-将config-service和admin-service注册到单独的eureka-server上)章节
>
> > 注3：在多机房部署时，往往希望config service和admin service只向同机房的eureka注册，要实现这个效果，需要利用`ServerConfig`表中的cluster字段，config service和admin service会读取所在机器的`/opt/settings/server.properties`（Mac/Linux）或`C:\opt\settings\server.properties`（Windows）中的idc属性，如果该idc有对应的eureka.service.url配置，那么就只会向该机房的eureka注册。比如config service和admin service会部署到`SHAOY`和`SHAJQ`两个IDC，那么为了实现这两个机房中的服务只向该机房注册，那么可以在`ServerConfig`表中新增两条记录，分别填入`SHAOY`和`SHAJQ`两个机房的eureka地址即可，`default` cluster的记录可以保留，如果有config service和admin service不是部署在`SHAOY`和`SHAJQ`这两个机房的，就会使用这条默认配置。
>
> | Key                | Cluster | Value                         | Comment              |
> | ------------------ | ------- | ----------------------------- | -------------------- |
> | eureka.service.url | default | <http://1.1.1.1:8080/eureka/> | 默认的Eureka服务Url  |
> | eureka.service.url | SHAOY   | <http://2.2.2.2:8080/eureka/> | SHAOY的Eureka服务Url |
> | eureka.service.url | SHAJQ   | <http://3.3.3.3:8080/eureka/> | SHAJQ的Eureka服务Url |
>
> ##### 2. namespace.lock.switch - 一次发布只能有一个人修改开关，用于发布审核
>
> 这是一个功能开关，如果配置为true的话，那么一次配置发布只能是一个人修改，另一个发布。
>
> > 生产环境建议开启此选项
>
> ##### 3. config-service.cache.enabled - 是否开启配置缓存
>
> 这是一个功能开关，如果配置为true的话，config service会缓存加载过的配置信息，从而加快后续配置获取性能。
>
> 默认为false，开启前请先评估总配置大小并调整config service内存配置。
>
> ##### 4. item.key.length.limit - 配置项 key 最大长度限制
>
> 默认配置是128。
>
> ##### 5. item.value.length.limit - 配置项 value 最大长度限制
>
> 默认配置是20000。

##### ApolloPortalDb

![1559442905409](/ApolloBuild/1559442905409.png)

同样在`ApolloPortalDb`中也存在一个配置表，这里`superAdmin`就是你一开始登陆Apollo的用户名，根据需要更改，我在这里未做更改。

> ##### 1.apollo.portal.envs - 可支持的环境列表
>
> 默认值是dev，如果portal需要管理多个环境的话，以逗号分隔即可（大小写不敏感），如：
>
> ```
> DEV,FAT,UAT,PRO
> ```
>
> ##### 2.organizations - 部门列表
>
> Portal中新建的App都需要选择部门，所以需要在这里配置可选的部门信息，样例如下：
>
> ```
> [{"orgId":"TEST1","orgName":"样例部门1"},{"orgId":"TEST2","orgName":"样例部门2"}]
> ```
>
> ##### 3.superAdmin - Portal超级管理员
>
> 超级管理员拥有所有权限，需要谨慎设置。
>
> 如果没有接入自己公司的SSO系统的话，可以先暂时使用默认值apollo（默认用户）。等接入后，修改为实际使用的账号，多个账号以英文逗号分隔(,)。
>
> ##### 4.consumer.token.salt - consumer token salt
>
> 如果会使用开放平台API的话，可以设置一个token salt。如果不使用，可以忽略。
>
> ##### 5.wiki.address
>
> portal上“帮助”链接的地址，默认是Apollo github的wiki首页，可自行设置。
>
> ##### 6.admin.createPrivateNamespace.switch
>
> 是否允许项目管理员创建private namespace。设置为`true`允许创建，设置为`false`则项目管理员在页面上看不到创建private namespace的选项。[了解更多Namespace](https://github.com/ctripcorp/apollo/wiki/Apollo核心概念之"Namespace")
>
> ##### 7. configView.memberOnly.envs
>
> 只对项目成员显示配置信息的环境列表，多个env以英文逗号分隔。
>
> 对设定了只对项目成员显示配置信息的环境，只有该项目的管理员或拥有该namespace的编辑或发布权限的用户才能看到该私有namespace的配置信息和发布历史。公共namespace始终对所有用户可见。

### 获取安装包

#### 修改脚本

> 可以通过两种方式获取安装包：
>
> 1. 直接下载安装包
> 	- 从[GitHub Release](https://github.com/ctripcorp/apollo/releases)页面下载预先打好的安装包
> 	- 如果对Apollo的代码没有定制需求，建议使用这种方式，可以省去本地打包的过程
> 2. 通过源码构建
> 	- 从[GitHub Release](https://github.com/ctripcorp/apollo/releases)页面下载Source code包或直接clone[源码](https://github.com/ctripcorp/apollo)后在本地构建
> 	- 如果需要对Apollo的做定制开发，需要使用这种方式

建议使用源码构建，编辑源码`\apollo-master\scripts\build.sh`：

![源码构建](/ApolloBuild/1559443582207.png)

**红色部分：需要修改的部分**

Ⅰ：配置数据库地址以及用户名、密码；

>  注意：填入的用户需要具备对ApolloPortalDB和ApolloConfigDB数据的读写权限。

Ⅱ：配置环境地址，由于Apollo支持分布式多环境配置，可以将不同的环境配置在不同的服务器，也就拥有了不同的地址，可以在这里进行配置，若用不到可以直接注释掉，这里填的是Eureka地址，即`apolloi-configservice`的地址。如：在数据库配置中我们给`eureka.service.url`配置了`http://192.168.13.13:8080/eureka/`,那么这里红线部分就应该是`192.168.13.13：8080`。

#### 编译打包

完成脚本的修改后，就可以执行脚本进行编译、打包了，建议在Linux下进行编译打包，Windows下可以通过Git Bash打包。

##### 获取安装包

三个安装包分别：

位于`apollo-configservice/target/`目录下的`apollo-configservice-x.x.x-github.zip`；

位于`apollo-adminservice/target/`目录下的`apollo-adminservice-x.x.x-github.zip`；

位于`apollo-portal/target/`目录下的`apollo-portal-x.x.x-github.zip`。

### 部署Apollo服务端

#### 部署apollo-configservice

将对应环境的`apollo-configservice-x.x.x-github.zip`上传到服务器上，解压后执行scripts/startup.sh即可。如需停止服务，执行scripts/shutdown.sh。

#### 部署apollo-adminservice

将对应环境的`apollo-adminservice-x.x.x-github.zip`上传到服务器上，解压后执行scripts/startup.sh即可。如需停止服务，执行scripts/shutdown.sh。

#### 部署apollo-portal

将`apollo-portal-x.x.x-github.zip`上传到服务器上，解压后执行scripts/startup.sh即可。如需停止服务，执行scripts/shutdown.sh。

# 验证

访问`http://xxx,xxx,xxx,xxx:8070`打开Apollo页面，默认超级管理员用户名为`apollo`，密码为`admin`。

![Apollo客户端](/ApolloBuild/1559618071934.png)