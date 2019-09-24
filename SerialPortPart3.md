---
title: Qt上位机开发：串口控制模块开发
date: 2019-04-08 20:51:00
tags: [Qt, 上位机, 嵌入式]
layout: post
comments: true
toc: true
---


&emsp;&emsp;这个项目是18年初寒假开始的，初始想法是利用寒假时间学习Qt开发并且顺便把毕业设计做出来，后来毕业设计做出来了，这个项目也完成了，然后又投入了二战，所以分享开发的过程的Blog却一直没有时间写，现在收到了录取通知，又想起这个事情，可能思绪不太连贯了，但我还是想把这个事情坐下来。

## 创建串口调试界面

**串口调试器的主要需求：**

1. 显示来自串口接收到的信息；
2. 向串口发送中断；
3. 配置串口。

<!--more-->

**根据需求设计UI界面**

<div align=center>
    <img src="/SerialPortPart3/image1.jpg"
</div>



&emsp;&emsp;其中界面的设计主要用到了`Qlabel`、`QTextEdit`、`QCamboBox`、`QPushButton`,其中较为复杂的应该是QTextEdit组件的输出格式控制。



## 配置串口

&emsp;&emsp;由于Qt5内置了串口类，所以我们可以通过直接调用类方法进行串口的配置，这真的是一件十分Nice的事情，之前我也有通过Keil开发过C51的串口模块，相比较而言真的是能省不少心。

&emsp;&emsp;当用户通过LogIn界面登陆进入串口界面时，此时程序已经开始查找当前可用串口，显示在端口号处。

```C++
    //查找可用的串口
    foreach(const QSerialPortInfo &info, QSerialPortInfo::availablePorts())
    {
        QSerialPort serial;
        serial.setPort(info);
        if(serial.open(QIODevice::ReadWrite))
        {
            ui->combox->addItem(serial.portName());
            serial.close();
        }
    }
    //设置波特率下拉菜单默认显示第三项
    ui->baudrate->setCurrentIndex(3);
    //关闭发送按钮的使能
 //   ui->openport->setEnabled(false);
    qDebug()<<tr("界面设定成功！");
```



## 打开串口

&emsp;&emsp;点击<kbd>打开串口</kbd>，系统自动读取用户所设置的串口配置，这里需要注意的是虽然有校验位选项，但是我没有使用，所以直接使用了`serial->setParity(QSerialPort::NoParity)`。从逻辑上来讲，当用户以当前设置打开了串口，设置变不能再更改了，所以在点击<kbd>打开串口</kbd>后，应将`QCamboBox`关闭使能，然后将`QPushButton`的文字内容改成“关闭串口”。

```c++
serial = new QSerialPort;
        //设置串口名
        serial->setPortName(ui->combox->currentText());
        //打开串口
        serial->open(QIODevice::ReadWrite);
        serialInfo = serialInfo + "Mode: ReadWrite" + '\n';
        //设置波特率
        serial->setBaudRate(ui->baudrate->currentText().toInt());
        //设置数据位数
        switch(ui->databit->currentIndex())
        {
        case 5: serial->setDataBits(QSerialPort::Data5); break;
        case 6: serial->setDataBits(QSerialPort::Data6); break;
        case 7: serial->setDataBits(QSerialPort::Data7); break;
        case 8: serial->setDataBits(QSerialPort::Data8); break;
        default: break;
        }
        serialInfo = serialInfo + "Data Bit: " + ui->databit->currentText() + '\n';
        //设置奇偶校验
        switch(ui->conparebit->currentIndex())
        {
        case 0: serial->setParity(QSerialPort::NoParity); break;
        default: break;
        }
        serialInfo = serialInfo + "Conpare Bit" + ui->conparebit->currentText() + '\n';
        //设置停止位
        switch(ui->stopbit->currentIndex())
        {
        case 1: serial->setStopBits(QSerialPort::OneStop); break;
        case 2: serial->setStopBits(QSerialPort::TwoStop); break;
        default: break;
        }
        serialInfo = serialInfo + "Stop Bit" + ui->stopbit->currentText() + '\n';
        //设置流控制
        serial->setFlowControl(QSerialPort::NoFlowControl);
        serialInfo = serialInfo + "FlowControl: No flow control." + '\n';

        //关闭设置菜单使能
        ui->combox->setEnabled(false);
        ui->baudrate->setEnabled(false);
        ui->databit->setEnabled(false);
        ui->conparebit->setEnabled(false);
        ui->stopbit->setEnabled(false);
        ui->openport->setText(tr("关闭串口"));
        ui->send->setEnabled(true);

        //连接信号槽
        QObject::connect(serial, &QSerialPort::readyRead, this, &usr::Read_Data);

        log("[usr.cpp]-[on_openport_clicked()]: System Start.");
        log("[usr.cpp]-[on_openport_clicked()]" +'\n' + serialInfo);
```



## 发送中断并获取当前光照强度

&emsp;&emsp;发送中断是获取数据的最基本的方式，获取实时数据或者定时获取数据归根结底都是基于发送中断于定时发送中断实现的。得益于Qt5串口类的封装，发送中断变得很简洁。

```c++
//向单片机发送check
void usr::sendchk()
{
    QString chk = "check";
    serial->write(chk.toLatin1());
    log("[usr.cpp]-[sendchk()]: CHECK has been sent.");
}
```



## 定时监控

&emsp;&emsp;通过定时的向串口发送中断获取信息，来起到监控数据的功能。

```c++
void usr::on_startAutoControl_clicked()
{

    if(token == false)
    {
        qDebug()<<tr("token == false");
        ui->startAutoControl->setText(tr("暂停监控"));
        qDebug()<<tr("暂停监控");
        token = true;
        qDebug()<<tr("token赋值为true");
        on_autoControlSwitch_stateChanged(Qt::Checked);
        log("[usr.cpp]-[on_startAutoControl_clicked()]: Auto Control has been started.");
    }
    else if(token == true)
    {
        qDebug()<<tr("token == true");
        ui->startAutoControl->setText(tr("开始监控"));
        qDebug()<<tr("开始监控");
        token = false;
        qDebug()<<tr("token赋值为false");
        on_autoControlSwitch_stateChanged(Qt::Unchecked);
        log("[usr.cpp]-[on_startAutoControl_clicked()]: Auto Control has been stopped.");
    }
}  
```

&emsp;&emsp;定时功能使用了`QTimer类`，并且通过槽函数机制链接到`sendchk()函数`给串口发送中断。

```c++
//开启自动监控
void usr::on_autoControlSwitch_stateChanged(int arg1)
{
    if(arg1 == Qt::Checked)
    {
        qDebug()<<tr("timer == checked");       //设置定时器
        timer = new QTimer(this);
        timer->setInterval(20000);               //设置时间间隔
        connect(timer,SIGNAL(timeout()),this,SLOT(sendchk()));      //链接槽函数
        qDebug()<<tr("Timer has been inited!");

        timer->start(1000);     //开始
        log("[usr.cpp]-[on_autoControlSwitch_stateChanged]: QTimer has been started.");

    }
    if(arg1 == Qt::Unchecked)
    {
        qDebug()<<tr("arg1 == unchecked");
        timer->stop();                      //暂停定时器
        qDebug()<<tr("已暂停定时器");
        log("[usr.cpp]-[on_autoControlSwitch_stateChanged]: QTimer has been paused.");
    }
}
```

## 显示数据

&emsp;&emsp;显示数据是我在开发过程中花费时间最长的模块，原因是我不太明白Qt接收到串口信息的数据类型以及如何进行格式转换。串口接收到的数据是十六进制的信息，此处需要对其进行格式转换，但是不知为何我在使用`toInt()方法`时总是有Bug，不太明白为什么，如果有大佬知道的话，请评论区解答一下，谢谢~  
&emsp;&emsp;所以这里提供另一种解决方案。总的逻辑是这样，首先通过寄存器`buf`暂存从串口中读取到的数据，然后通过`bytesToInt(buf)`转化成整型数据，然后再通过`QString::number(decbuf,10)`将其转化成字符串型输出。

```c++
//将QByteArray型转换成int型
int usr::bytesToInt(QByteArray bytes)
{
    int addr = bytes[0] & 0x000000FF;
    addr |= ((bytes[1] << 8) & 0x0000FF00);
    addr |= ((bytes[2] << 16) & 0x00FF0000);
    addr |= ((bytes[3] << 24) & 0xFF000000);
    return addr;
}

//读取接收到的数据
void usr::Read_Data()
{
    QByteArray buf;
    int decbuf;
    //bool ok;

    //读取串口数据
    buf = serial->readAll();

    //对串口数据进行格式转换
    decbuf = bytesToInt(buf);
    QString strbuf = QString::number(decbuf,10);
    QString strTime = getTime();

    if(ifHandle == true)
    {
        strbuf = strTime + ":      光照强度为：" + strbuf + "**********手动获取" + '\n';
        writeFile(strbuf);
        ifHandle = false;
    }
    else
    {
        strbuf = strTime + ":      光照强度为：" + strbuf + "**********自动获取" + '\n';
        writeFile(strbuf);
    }

    if(!buf.isEmpty())
    {
        QString str = ui->outputEdit->toPlainText();

        str+=strbuf + '\n';
        ui->outputEdit->clear();
        ui->outputEdit->append(str);
    }

    buf.clear();
}
```



## 系统日志

&emsp;&emsp;这个模块不是必要模块，有没有都不影响程序运行，只是在开发过程调试Bug中大量使用了`qDebug()`打印运行信息，最后索性将其封装成一个单独的方法，用来保存运行信息，当系统宕机时，可以查看系统日志了解系统在哪个模块出现了问题。

```c++
//系统日志
void usr::log(QString str)
{
    QFile file("log.csv");
    QString strTime = getTime();

    str = '\n' + strTime + ":  " + str;

    file.open(QIODevice::ReadWrite | QIODevice::Append);

    QTextStream out(&file);
    out<<str<<endl;
    qDebug()<<"write success";
}
```

## 数据存储

&emsp;&emsp;数据存储讲道理是应该架在数据库上的，但是我为了图方便没有使用使用数据库，只做了一张表格存储数据。说到这，上个月我去参加山大的研究生复试，面试的时候介绍我的项目，因为这个还被老师质疑了。

> 我：………………（介绍我的项目）……老师，我介绍完了。
> 老师：就这个？
> 我：？？？
> 我：就这个...
> 老师：你用了什么数据库？
> 我：没用数据库...
> 老师：没用数据库？
> 我：对。
> 老师：行，就这样吧。

&emsp;&emsp;所以说这东西还有很多东西需要继续做，如果大家有兴趣可以继续做下去。

```c++
//写文件模块
void usr::writeFile(QString str)
{
    QFile file("data.csv");
    QDir dir;
    qDebug()<<dir.currentPath();

    file.open(QIODevice::ReadWrite | QIODevice::Append);

    QTextStream out(&file);
    out<<str<<endl;
    qDebug()<<"write success";
    log("[usr.cpp]-[writeFile(QString str)]： write success");
}
```

## 总结

&emsp;&emsp;这个项目本身难度不算太高，跟着这三篇Blog自己单独的做下来我认为问题不大，但这恰恰失去了Debug本身的乐趣，我至今还记得解决每一个问题时的喜悦心情，那种欢愉是copy难以企及的。
&emsp;&emsp;我认为程序猿最重要的学习方式就是在Coding过程中你所收获的新的解题思路以及欢愉。

## 附录

单片机代码
`main.c`

```c
#include "led.h"
#include "delay.h"
#include "key.h"
#include "sys.h"
#include "lcd.h"
#include "usart.h"	 
#include "adc.h"

 extern u8 send_flag;
 int main(void)
 {	 
   	u16 adcx;
	float temp;
	delay_init();	    	 //延时函数初始化	  
	NVIC_Configuration(); 	 //设置NVIC中断分组2:2位抢占优先级，2位响应优先级
	uart_init(9600);	 	//串口初始化为9600
 	LED_Init();			     //LED端口初始化 	
 	Adc_Init();		  		//ADC初始化
	while(1)
	{
		if(send_flag == 1 )
		{
			send_flag = 0;
			adcx=Get_Adc_Average(ADC_Channel_1,10);
			USART_SendData(USART1,adcx);
		
		}
	}
 }

```

`usart.c`

```c
#include "sys.h"
#include "usart.h"	  
////////////////////////////////////////////////////////////////////////////////// 	 
//如果使用ucos,则包括下面的头文件即可.
#if SYSTEM_SUPPORT_UCOS
#include "includes.h"					//ucos 使用	  
#endif

u8 send_flag = 0;
u8 rev_c_flag = 0;
u8 rev_h_flag = 0;
u8 rev_e_flag = 0;
u8 rev_c2_flag = 0;
u8 rev_k_flag = 0;
#if 1
#pragma import(__use_no_semihosting)             
//标准库需要的支持函数                 
struct __FILE 
{ 
	int handle; 

}; 

FILE __stdout;       
//定义_sys_exit()以避免使用半主机模式    
_sys_exit(int x) 
{ 
	x = x; 
} 
//重定义fputc函数 
int fputc(int ch, FILE *f)
{      
	while((USART1->SR&0X40)==0);//循环发送,直到发送完毕   
    USART1->DR = (u8) ch;      
	return ch;
}
#endif 

#if EN_USART1_RX   //如果使能了接收
//串口1中断服务程序
//注意,读取USARTx->SR能避免莫名其妙的错误   	
u8 USART_RX_BUF[USART_REC_LEN];     //接收缓冲,最大USART_REC_LEN个字节.
//接收状态
//bit15，	接收完成标志
//bit14，	接收到0x0d
//bit13~0，	接收到的有效字节数目
u16 USART_RX_STA=0;       //接收状态标记	  

//初始化IO 串口1 
//bound:波特率
void uart_init(u32 bound){
    //GPIO端口设置
    GPIO_InitTypeDef GPIO_InitStructure;
	USART_InitTypeDef USART_InitStructure;
	NVIC_InitTypeDef NVIC_InitStructure;
	 
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1|RCC_APB2Periph_GPIOA, ENABLE);	//使能USART1，GPIOA时钟
 	USART_DeInit(USART1);  //复位串口1
	 //USART1_TX   PA.9
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9; //PA.9
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;	//复用推挽输出
    GPIO_Init(GPIOA, &GPIO_InitStructure); //初始化PA9
   
    //USART1_RX	  PA.10
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING;//浮空输入
    GPIO_Init(GPIOA, &GPIO_InitStructure);  //初始化PA10

   //Usart1 NVIC 配置

    NVIC_InitStructure.NVIC_IRQChannel = USART1_IRQn;
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority=3 ;//抢占优先级3
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 3;		//子优先级3
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;			//IRQ通道使能
	NVIC_Init(&NVIC_InitStructure);	//根据指定的参数初始化VIC寄存器
  
   //USART 初始化设置

	USART_InitStructure.USART_BaudRate = bound;//一般设置为9600;
	USART_InitStructure.USART_WordLength = USART_WordLength_8b;//字长为8位数据格式
	USART_InitStructure.USART_StopBits = USART_StopBits_1;//一个停止位
	USART_InitStructure.USART_Parity = USART_Parity_No;//无奇偶校验位
	USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;//无硬件数据流控制
	USART_InitStructure.USART_Mode = USART_Mode_Rx | USART_Mode_Tx;	//收发模式

    USART_Init(USART1, &USART_InitStructure); //初始化串口
    USART_ITConfig(USART1, USART_IT_RXNE, ENABLE);//开启中断
    USART_Cmd(USART1, ENABLE);                    //使能串口 

}

void USART1_IRQHandler(void)                	//串口1中断服务程序
	{
	u8 Res;
#ifdef OS_TICKS_PER_SEC	 	//如果时钟节拍数定义了,说明要使用ucosII了.
	OSIntEnter();    
#endif
	if(USART_GetITStatus(USART1, USART_IT_RXNE) != RESET)  //接收中断(接收到的数据必须是0x0d 0x0a结尾)
		{
		Res =USART_ReceiveData(USART1);//(USART1->DR);	//读取接收到的数据
		if((rev_c_flag == 0)&&(rev_h_flag == 0)&&(rev_e_flag == 0)&&(rev_c2_flag == 0)&&(rev_k_flag == 0))
		{
			if(Res == 'c')
			{
				rev_c_flag = 1;
			}
		}
		if((rev_c_flag == 1)&&(rev_h_flag == 0)&&(rev_e_flag == 0)&&(rev_c2_flag == 0)&&(rev_k_flag == 0))
		{
			if(Res == 'h')
			{
				rev_h_flag = 1;
			}
		}
		if((rev_c_flag == 1)&&(rev_h_flag == 1)&&(rev_e_flag == 0)&&(rev_c2_flag == 0)&&(rev_k_flag == 0))
		{
			if(Res == 'e')
			{
				rev_e_flag = 1;
			}
		}
		if((rev_c_flag == 1)&&(rev_h_flag == 1)&&(rev_e_flag == 1)&&(rev_c2_flag == 0)&&(rev_k_flag == 0))
		{
			if(Res == 'c')
			{
				rev_c2_flag = 1;
			}
		}
		if((rev_c_flag == 1)&&(rev_h_flag == 1)&&(rev_e_flag == 1)&&(rev_c2_flag == 1)&&(rev_k_flag == 0))
		{
			if(Res == 'k')
			{
				rev_c_flag = 0;
				rev_h_flag = 0;
				rev_e_flag = 0;
				rev_c2_flag = 0;
				rev_k_flag = 0;
				send_flag = 1;
			}
		}
		
     } 
#ifdef OS_TICKS_PER_SEC	 	//如果时钟节拍数定义了,说明要使用ucosII了.
	OSIntExit();  											 
#endif
} 
#endif	
```

## 源码下载

[Github](https://github.com/GiottoLee/SerialPort)