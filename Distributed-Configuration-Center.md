---
title: 基于Docker+Gitlab搭建Spring Cloud分布式配置中心
date: 2019-05-23 14:11:18
tags: [Docker,Gitlab,Spring Cloud,分布式配置中心,Help]
toc: true
comment: true
---

# 分布式配置中心

在分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。在 Spring Cloud 中，有分布式配置中心组件 Spring Cloud Config ，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程 Git 仓库中。在 Spring Cloud Config 组件中，分两个角色，一是 Config Server，二是 Config Client。

<!--more-->

# 分布式配置中心服务端

## 概述

创建一个工程名为 `spring-cloud-demo-config` 的项目，`pom.xml` 配置文件如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>spring-cloud-demo-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../spring-cloud-demo-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>spring-cloud-demo-config</artifactId>
    <packaging>jar</packaging>

    <name>spring-cloud-demo-config</name>
    <url>http://www.funtl.com</url>
    <inceptionYear>2018-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zipkin</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.spring.cloud.demo.config.ConfigApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

主要增加了 `spring-cloud-config-server` 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

## Application

通过 `@EnableConfigServer` 注解，开启配置服务器功能

```java
package com.funtl.hello.spring.cloud.config;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableConfigServer
@EnableEurekaClient
public class ConfigApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigApplication.class, args);
    }
}
```

## application.yml

增加 Config 相关配置，并设置端口号为：`8888`

```yml
spring:
  application:
    name: spring-cloud-demo-config
  cloud:
    config:
      label: master
      server:
        git:
          uri: http://192.168.***.***/GiottoLee/spring-cloud-config.git
          search-paths: respo
          username: GiottoLee
          password: 12345678

server:
  port: 8888

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

相关配置说明，如下：

- `spring.cloud.config.label`：配置仓库的分支
- `spring.cloud.config.server.git.uri`：配置 Git 仓库地址（GitHub、GitLab、码云 ...）
- `spring.cloud.config.server.git.search-paths`：配置仓库路径（存放配置文件的目录）
- `spring.cloud.config.server.git.username`：访问 Git 仓库的账号
- `spring.cloud.config.server.git.password`：访问 Git 仓库的密码

注意事项：

- 如果使用 GitLab 作为仓库的话，`git.uri` 需要在结尾加上 `.git`，GitHub 则不用

## 测试

浏览器端访问：http://localhost:8888/config-client/dev/master 显示如下：

```xml
<Environment>
<name>config-client</name>
<profiles>
<profiles>dev</profiles>
</profiles>
<label>master</label>
<version>7ccbf2cf44d5c6607d4d593659238cb95edba2db</version>
<state/>
<propertySources/>
</Environment>
```

证明配置服务中心可以从远程程序获取配置信息

## 附：HTTP 请求地址和资源文件映射

- http://ip:port/{application}/{profile}[/{label}]
- http://ip:port/{application}-{profile}.yml
- http://ip:port/{label}/{application}-{profile}.yml
- http://ip:port/{application}-{profile}.properties
- http://ip:port/{label}/{application}-{profile}.properties

# 分布式配置中心客户端

在服务消费者（ribbon或feign）的`pom.xml`中添加 `spring-cloud-starter-config` 依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

## 服务消费者

入口类没有需要特殊处理的地方，代码如下：

```java
package com.funtl.hello.spring.cloud.config.client;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class WebAdminFeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(WebAdminFeignApplication.class, args);
    }
}
```

## application.yml

Feign原配置文件为：

```java
spring:
  application:
    name: spring-cloud-demo-web-admin-feign
  thymeleaf:
    cache: false
    mode: LEGACYHTML5
    encoding: UTF-8
    servlet:
      content-type: text/html
  zipkin:
    base-url: http://localhost:9411

server:
  port: 8765

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

feign:
  hystrix:
    enabled: true
```



现在将其移入`spring-cloud-config`配置文件夹，重命名为`web-admin-feign-dev.yml`，并增加新的`application.yml`如下 ：

```yml
spring:
  cloud:
    config:
      uri: http://localhost:8888
      name: web-admin-feign
      label: master
      profile: dev
```

相关配置说明，如下：

- `spring.cloud.config.uri`：配置服务中心的网址

- `spring.cloud.config.name`：配置文件名称的前缀

- `spring.cloud.config.label`：配置仓库的分支

- `spring.cloud.config.profile`：配置文件的环境标识
- dev：表示开发环境
	- test：表示测试环境
- prod：表示生产环境

**实际使用中只需更改`profile`的标识（dev、test、prod）即可变更运行环境配置**

注意事项：

- 配置服务器的默认端口为 `8888`，如果修改了默认端口，则客户端项目就不能在 `application.yml` 或 `application.properties` 中配置 `spring.cloud.config.uri`，必须在 `bootstrap.yml` 或是 `bootstrap.properties` 中配置，原因是 `bootstrap` 开头的配置文件会被优先加载和配置，切记。

## 附：开启 Spring Boot Profile

我们在做项目开发的时候，生产环境和测试环境的一些配置可能会不一样，有时候一些功能也可能会不一样，所以我们可能会在上线的时候手工修改这些配置信息。但是 Spring 中为我们提供了 Profile 这个功能。我们只需要在启动的时候添加一个虚拟机参数，激活自己环境所要用的 Profile 就可以了。

操作起来很简单，只需要为不同的环境编写专门的配置文件，如：`application-dev.yml`、`application-prod.yml`， 启动项目时只需要增加一个命令参数 `--spring.profiles.active=环境配置` 即可，启动命令如下：

```bash
mvn clean package      //打成jar包
java -jar hello-spring-cloud-web-admin-feign-1.0.0-SNAPSHOT.jar --spring.profiles.active=prod
```