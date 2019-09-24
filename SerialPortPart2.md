---
title: Qt上位机开发：管理员登陆模块开发
date: 2018-01-22 15:20:00
tags: [Qt, 上位机, 嵌入式]
layout: post
comments: true
toc: true
---

# 1.Create Project

&emsp;&emsp;从今天开始，我们开始我们的上位机开发大师之路。

&emsp;&emsp;首先我们要先创建我们的第一个工程。文件->新建文件或项目或者使用快捷键Ctrl+N。

<div align=center>
    <img src="/SerialPortPart2/image1.png">
</div>

&emsp;&emsp;这里我们选择Application->Qt WidgetsApplication。

<!--more-->

<div align=center>
    <img src="/SerialPortPart2/image2.png">
</div> 

&emsp;&emsp;下一步之后我们要给我们的项目起一个名字，这里我起名叫做serialport，选择我们的项目路径，需要注意的是项目路径这里不允许有中文，如果有中文项目构建过程中会报错。

<div align=center>
    <img src="/SerialPortPart2/image3.png">
</div> 

&emsp;&emsp;下一步选择编译器，由于在昨天的环境搭建中，我只安装了MinGw一种编译环境，所以这里也只有这一个Desktop Qt 5.10.0 MinGW32 bit，如果你在之前的安装过程中还选择了其他编译器的话，这里会显示多个编译器。

 <div align=center>
    <img src="/SerialPortPart2/image3.png">
</div>

&emsp;&emsp;下一步，基类我们选择QWidget，类名改为login。原因是因为我想在串口调试助手的基础上添加管理员登陆的安全验证，所以我们以login作为我们的初始界面。（这里我忘记改了-。-||）

<div align=center>
    <img src="/SerialPortPart2/image5.png">
</div> 

&emsp;&emsp;创建完成后我们发现Qt自动生成了cpp文件和头文件，到这一步我们的项目初创成功。

# 2.创建多窗口

&emsp;&emsp;双击Forms文件夹下的ui文件，我们在用户界面上添加我们需要用到的组件。

<div align=center>
    <img src="/SerialPortPart2/image6.png">
</div>

&emsp;&emsp;我要实现的功能是当用户输入正确的口令，点击Log in时，我们这个程序才会跳转到串口工具的界面，点击Cancel直接退出程序。

## 2.1 Cancel功能

&emsp;&emsp;我们从最简单的开始，实现Cancel的关闭功能。

&emsp;&emsp;实现关闭功能，不需要敲代码，我们只需要在Signals & Slots Editor手动选择一下就好了。

<div align=center>
    <img src="/SerialPortPart2/image7.png">
</div>

​	发送者是Cancel这个pushButton的对象名，信号选择clicked（），接收者选择Widget，槽函数选择close（）。这样就实现了Cancel功能。

## 2.2  Log in功能
&emsp;&emsp;我们要实现登陆功能，首先我们要另外创建一个界面（usr），当我们从Login界面输入账号密码，验证正确后，即可进入usr界面操作，所以我们先来看一下创建新界面的步骤。

&emsp;&emsp;文件->新建文件或项目

<div align=center>
    <img src="/SerialPortPart2/image8.png">
</div>

&emsp;&emsp;这个类可以帮助我们在创建窗口程序的基础上将cpp文件和头文件一起创建出来。

&emsp;&emsp;之后，选择Dialogwithout Buttons，按键什么的我们自己添加就可以了。

<div align=center>
    <img src="/SerialPortPart2/image9.png">
</div>

<div align=center>
    <img src="/SerialPortPart2/image10.png">
</div>

&emsp;&emsp;然后我们起个名，就叫usr好了。

&emsp;&emsp;之后下一步、下一步…就可以完成创建了。

# 3.用户验证

&emsp;&emsp;下面我们来做用户验证的部分，从现在开始就要开始敲代码了，右键Login按键，选择转到槽，然后选择clicked（），按键事件的函数Qt就为我们自动创建好了。

&emsp;&emsp;首先我们要在login.h里创建usr窗口的对象。

```c++
#ifndef LOGIN_H
#define LOGIN_H

#include <QWidget>
#include "usr.h"

namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

private slots:
    void on_pushButton_clicked();

private:
    Ui::Widget *ui;
    usr *Usr=new usr;

};

#endif // LOGIN_H
```

&emsp;&emsp;接下来的代码要执行的操作就是验证账号密码，如果正确则跳转usr窗口，如果不正确，发出警告。另外，我们不管平时用QQ还是用微信登陆，密码都是不显示的小黑点，这里我们也设置一下。

```c++
#include "login.h"
#include "ui_login.h"
#include "QMessageBox"
 
Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
    ui->password->setEchoMode(QLineEdit::Password);//当输入密码时，显示为*******
}
 
Widget::~Widget()
{
    delete ui;
}
 
void Widget::on_pushButton_clicked()
{
/*
    this->hide();
    Usr->show();
*/
 
 
    if(this->ui->usrname->text().trimmed() == tr("admin") &&
           this->ui->password->text().trimmed()== tr("root"))  //去除lineEdit内的用户名和密码进行校验
        {
            //登陆成功后显示对话框
            this->hide();
            Usr->show();
 
 
        }
        else
        {
            //用户输入存在错误
            QMessageBox::warning(this,tr("waring"),tr("your passward is wrong"),QMessageBox::Yes);
            ui->usrname->clear();  //清空姓名usrname
            ui->password->clear();  //清空密码passward
            ui->usrname->setFocus();  //将鼠标重新定位到usrname
        }
}
```

&emsp;&emsp;我们Ctrl+B构建运行一下

<font color="pink">

插入栓插入...
解放播放传导系统 准备接续...
探针插入 完毕
神经同调装置在基准范围内
第一次接触...
插入栓注水...
主电源连接完毕...
开始进行第二次接触...
交互界面连接...
思考形态以中文作为基准，进行思维连接...
连接没有异常
同步率为 1000.0000%%
第一锁定器解除...
第二锁定器解除...
移往播放口...

</font>

&emsp;&emsp;密码正确--->跳转usr窗口

<div align=center>
    <img src="/SerialPortPart2/image11.png">
</div>


&emsp;&emsp;密码错误--->警告

 <div align=center>
     <img src="/SerialPortPart2/image12.png">
 </div>




&emsp;&emsp;至此，登陆界面完成！



---------------------