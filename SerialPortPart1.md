---

title: Qt上位机开发：Qt5的环境搭建
date: 2018-01-21 17:55:21
tags: [Qt, 上位机, 嵌入式]
layout: post
comments: true
toc: true
---



# 1. 序言

&emsp;&emsp;今年马上就要毕业，开始着手准备自己的毕业设计，当时一时脑热毕设方向选择了嵌入式设备开发，终端部分已经基本做好了，但是却始终找不到适应我需求的串口调试助手，所以决定自己撸一下Qt，自己写一个上位机，同时把开发过程写在这里，供大家参考指正。

# 2. Qt版本的选择

&emsp;&emsp;Qt的版本种类比较多，我选择的是Qt5的版本，原因是因为Qt5自带串口类，Qt以前的版本中，没有提供官方的对RS232串口的支持，编写串口程序很不方便。而Qt5版本中，官方自带了Qserialport模块，这极大地方便了我们的开发过程。

&emsp;&emsp;就Qt的编译环境也有两种，一种是MinGw，一种是MSVC。

&emsp;&emsp;MinGW这种编译环境是在Qt的安装包中捆绑安装配置好的，属于傻瓜式配置过程，而MSVC需要在安装Qt后，

&emsp;&emsp;再安装Visual Studio C++Compiler，过程并比较复杂，如无特殊需求，我认为选择MinGW版本就好。



<!--more-->



# 3. 环境搭建

## 3.1 Qt的下载

&emsp;&emsp;Qt我推荐是从官网（<https://www.qt.io/download>）下载。

&emsp;&emsp;选择OpenSource下载。

<div align=center>
    <img src="/SerialPortPart1/image1.png">
</div>


&emsp;&emsp;此时会要求你填写一些信息和注册一个账号，就随便填一填就可以下载了。在这里要吐槽一下这里要求注册填写的密码，密码要求至少含有三种格式的字符，要有大写、小写还有标点符号，简直反人类-。-+

&emsp;&emsp;但是这里注册的账号密码不要忘记，安装的过程中我们还要用。

## 3.2 Qt的安装

&emsp;&emsp;下载完之后安装包的大小是2.3G，作为一个安装包，还是挺大的了。



<div align=center>
    <img src="/SerialPortPart1/image2.png">
</div>


&emsp;&emsp;我们双击打开，首先肯定是例行的安全警告，毫无疑问看都不看直接确定。

&emsp;&emsp;看一下版本是最新的5.10，嗯，没错->下一步~

 <div align=center>
    <img src="/SerialPortPart1/image3.png">
</div>

&emsp;&emsp;到了这一步就要用到我们之前注册的账号，登陆一下然后下一步~需要注意的是这一步有一定的几率会提示你无法connect，倒杯茶休息一会，再试试就差不多可以了。

<div align=center>
    <img src="/SerialPortPart1/image4.png">
</div>

&emsp;&emsp;然后是设置你的安装路径，你开心就好，没什么可说的，不过需要注意的是我在使用Qt开发的过程中发现，Qt在对项目构建的时候，如果项目路径存在中文，就会出bug，我不太确定Qt的安装路径出现中文会怎样，不过为了一劳永逸，尽量还是不要有中文吧。

<div align=center>
    <img src="/SerialPortPart1/image5.png">
</div>

&emsp;&emsp;这一步十分重要！十分重要！我就不说三遍！

<div align=center>
    <img src="/SerialPortPart1/image6.png">
</div>

&emsp;&emsp;这个安装包类似Visual Studio的安装过程，可以自己选择性的安装工具，这里我选择了MinGW 5.3.0 32bit和Sources。

<div align=center>
    <img src="/SerialPortPart1/image7.png">
</div>

&emsp;&emsp;在Tools里我选择了MinGW 5.3.0，我们只选择这三项就足够了，我第一次安装中把所有的插件都安装了，安装完大约是12G，这里我觉得没有必要，安装了多种编译环境，在开发时还会经常需要选择编译器，有时候还会提示qmake.exe出错，我不太清楚什么原因，我猜是因为它把编译器搞混了，也可能是因为我没有手动设置（欢迎大佬指正）。

&emsp;&emsp;之后就是下一步、下一步……直到安装完成，安装速度挺慢的，耐心等一下就好。

&emsp;&emsp;明天我们正式开始上位机开发。







