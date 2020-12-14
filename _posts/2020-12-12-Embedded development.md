---
layout:     post
title:      "Embedded development base on itop4412"
date:       2020-12-12 16:45:00
author:     "Alanby"
header-img: "img/post-seu-build01.jpg"
tags:
    - Embedded
    - Linux
    - 
---



# Embedded development



## 1.需求分析

### **1.1 基本需求**

1) dada
1.dadadad
+数据采集模块每 0.5 秒采集一次数据，AD 读取阻值、摄像头获取图片； 
*数据通讯模块数据传输实现双向通讯，上位机查询，下位机应答，每上传一帧数据，上 位机反馈数据传输结果（成功、失败-接收错误或超时，错误码等）； 
- 本地监控界面能够在触摸屏上显示阻值、图片和报警信息； 
- 上位机监控界面能够在上位机实时显示阻值、图片和报警信息。



### 1.2 拔高需求

-  数据采集模块采集时间可以设置（0.1s-5s）； 

- 数据通讯模块加入通讯心跳包（1s），连续三次则判定通讯故障； 

- 本地监控界面历史数据，能显示阻值历史曲线，图片列表等； 
- 上位机监控界面历史数据，能显示阻值历史曲线，图片列表、终端状态（在线、离线）。



### 1.3 进阶需求

- 数据采集模块电阻上传时间：0.1s-5s 可设置，图片，动态监控，监测到有移动物体，则上传图片； 
- 数据通讯模块实现以太网 MQTT 协议； 
- 本地监控界面 UI 设计美观，登陆界面（本地输入，远程验证，输入时密码显示 *），等 其他附加功能； 
- 上位机监控界面 UI 设计美观，登陆界面（本地输入，远程验证，密码等其他附加功能。



## 2. 开发平台

### 2.1 硬件平台

- itop4412 开发板 
- Ubuntu20.04 操作系统的 PC 一台 
- CAT5E 网线一根 
- OV5640 摄像头一个

### 2.2  软件平台

- 利用 tftp 在上下位机之间传输文件 
- 虚拟机 VMware Workstation Pro 
- MobaXterm 
- QT Creator



## 3. 总体方案设计

### 3.1 开发环境搭建

开发环境搭建主要是针对 Ubuntu20.04 下的环境搭建和 window10 下的文件交互搭建。

#### 3.1.1 Windows系统

其中 window10 下需要利用软件 MobaXterm 和基于 UDP 的 tftp 文件传输软件，分别作为操作开发板的终端和在开发板与上位机之间文件传输的工具。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Linux_ITOP_01.png)



#### 3.1.2 Linux系统

主要是 Qt_Creator 的配置和虚拟网络的配置以及 arm-linux-gcc 编译环境的配置和 qmake 编译环境的配置，其中虚拟网络的配置是为了让开发板和上位机之间能相互 ping 通， 即可利用 TCP/IP 互相传输数据，而 QT 则是为了在上位机设计界面。Qt 的在 linux 下的 安装只需要执行两条指令即可：

```
sudo apt-get install build-essential
sudo apt-get install cmake qt5-default qtcreator

```

对于虚拟网络的配置，主要有以下几个步骤：

- 打开虚拟机左上角虚拟网络编辑器，使用管理员权限更改 VMNbet0 为桥接模式，并 选择直连网卡；
- 接着点击右上角虚拟机，设置中点击网络适配器，选择自定义，并下拉菜单选择 VMNet0；
- 在虚拟机右上交 setting 中，选择网络，把有线打开，在 IPV4 中，选择手动，地址填 与开发板同一个网段即可互相 ping 通。

对于 arm-linux 和 qmake 编译环境的配置, 输入指令较多，比较复杂。

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Linux_ITOP_02.png)



### 3.2 方案设计

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Linux_ITOP_03.png)



#### 3.2.1 数据采集

数据采集部分，基础、提高和进阶要求的方案设计如下：

- AD 读取和摄像头拍照：这里利用实验一中的 AD 读值和实验五的电子相册即可；
- 采集时间可设置：在 QT 中，采集时间取决去多长时间执行一起读取 AD 口的函数，可 以利用 QT 中的 QTimer 定时器，对 AD 读取的和拍照的内存读取函数按一定的时间 Data_timer 执行，另外加一个下拉菜单，时期 0.1-5s 可选，通过选择改变 Data_timer 的值，从而改变采集时间；
- 动态检测：这里是检测有移动的物体，选择了差帧法，利用前后两帧的图像矩阵做差 并将差值的绝对值相加，得到差值和，若大于一定阈值，则可说明图像中物体有变化， 此时可拍照上传。

#### 3.2.2 数据通讯

基本功能与拔高功能方案设计如下：

- 双向通讯：双向通讯使用 TCP/IP 实现，当上位机每接收到一张照片时，向下位机发 送一条接收成功的信息；
- 通讯心跳包：下位机通过 QT 定时器，每间隔 1s 向上位机发送一条固定的信息，说 明下位机仍然在执行中，上位机每隔 3s 查看下位机是否发送来了信息，如果超过 3s， 没接收到下位机的信息，则上位机自动断开连接，说明下位机已经停止。

#### 3.2.3 本地监控

基础、拔高和进阶要求如下：

- 显示信息：显示阻值、报警和图片信息只需要简单的界面控件即可；
- 绘制曲线：这里有两种方案，一种是使用第三方的曲线包，稍微重载一下函数即可；另 外一个方案是直接利用 QTpaint 和 QTpen 等工具，自己绘制出一个曲线；
- UI 界面：登陆界面可以使用外加一个界面覆盖来完成，其中密码等功能可使用字符 串数组拼接的方式完成。登陆密码错误时，也会弹出 messagebox，要求用户重新输入。

#### 3.2.4 上位机监控

上位机界面与下位机思路基础一致，这里主要说说如何读取图片和阻值：

- 阻值：首先将下位机发送来的阻值信息存在一个 txt 文件夹中，相当于缓存区域，然 后再从文件夹中读取阻值并显示；
- 图片读取：当接收到一张图片时，存到一个特定路径的文件夹中，通过扫描文件夹是 否增加了新的行，如果增加，说明接收到了新图片，并且将其显示在上位机监控界面 中。



## 4. 设计分工

课程设计分工主要为下位机和上位机开发以及最后一起处理通讯问题。下图是整体完成情况:

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Linux_ITOP_04.png)

### 4.1 下位机开发

下位机开发任务主要有三大块，包括登陆界面和主菜单界面设计、数据采集和绘制曲线模块。整体框图如下：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Linux_ITOP_05.png)



#### 4.1.1 界面设计

这部分主要采用了一个嵌套结构，首先是登陆界面，由于没有 USB 键盘，所以在界面右侧 做了一个 0-9 数字的小键盘按钮，供输入密码，密码正确后直接进入主菜单界面，两个界面 的切换时利用 signal 机制来完成的。核心代码如下：

```c
w = new Qt1();
6
w->show();
login login_widgt;
// 绑 定 登 陆 界 面 的signal函 数 到 主 界 面
QObject::connect(\&login_widgt , SIGNAL(sig_login()), w, SLOT(show()));
// 首 先 显 示 登 陆 界 面
login_widgt.show();
// 登 陆 完 成 后 关 闭， 显 示 主 界 面
QObject::connect(\&login_widgt ,SIGNAL(sig_login()),\&login_widgt ,SLOT(close()));
```

效果图为：

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Linux_ITOP_06.png)



#### 4.1.2 数据采集

这部分主要有动态监测和调整采集时间的功能，其中动态检测使用差帧法，利用前后两帧数 组对应位置像素值相减的绝对值求和，大于一定数值就认为是有动态变化。核心代码如下：

```c
void Is\_Same()
{
    int i = 0,sum = 0 ,err = 0;
    for(i=0; i<(320*240*2);i++)
    {
        // 前 一 帧 减 当 前 帧
        err = pre\_pic[i] - now\_pic[i];
        sum += abs(err);
    }
    if(sum > 30) flag\_same = 0; // 两 帧 不 同
    else flag\_same = 1; // 两 帧 相 同
}
```

另一部分是调整采集时间，这部分相对简单一点，利用 QT 定时器，设定一定的时间 执行采集函数即可。为了时间可调，在界面中可以加入 comboBox，也就是具有下拉选择功 能的控件，然后读取控件的值，赋予定时器即可。

```c
void Qt1::time_s()
{
    // pub_adc_peroid为 采 集 和 发 布 时 间
    if (comboBox ->currentIndex() == 0) pub_adc_peroid = 100;
    else if (comboBox ->currentIndex() == 1) pub_adc_peroid = 200;
    else if (comboBox ->currentIndex() == 2) pub_adc_peroid = 300;
    else if (comboBox ->currentIndex() == 3) pub_adc_peroid = 400;
    else pub_adc_peroid = 500;
}
```



#### 4.1.3 曲线绘制

下位机的曲线绘制没有用第三方包，用 QT 本身自带的库 QPainter、OPointF 和 QPen。 坐标轴用坐标背景用 setwindow 绘制，painter.drawLine 绘制，坐标轴中的点信息等用 painter.drawText 绘制，电阻值之间用直接连接，每次刷新，当刷新到固定点数时，也就 是当前画面填满时，清空当前坐标中的阻值，重新绘制，并将其保存。

```C
void chart::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
    QPen pen;
    painter.setWindow(-100, -20, 200, 40); // 设 置 界 面
    painter.fillRect(-100, -20, 200, 40, Qt::white); // 填 充 界 面
    int Xline = -100;
    for(int i=1;i<=10;i++) // 坐 标 轴
    {
    painter.drawLine(QPointF(Xline, 18.5), QPointF(Xline+20, 18.5));
    painter.drawLine(QPointF(Xline, 18.3), QPointF(Xline, 18.7));
    Xline+=20;
    }
    QFont font("Microsoft YaHei",3,QFont::Normal ,true); // 字 体 设 置
    painter.setFont(font);
    painter.setPen(Qt::red);
    painter.drawText(Xline ,15,QString::number((int)(Xline))); // 文 本 设 置
    pen.setColor(Qt::yellow); // 设 置 画 笔 颜 色
    pen.setWidthF(0.4);
    painter.setPen(pen);
}
```

![image](https://github.com/Alanaab/Alanaab.github.io/raw/master/img/Linux_ITOP_07.png)

### 4.2 上下位机通信

主要为 TCP/IP 通信方式，利用 TCP 上传阻值，FTP 传输图片。

#### 4.2.1 传输图片

首先启动服务器，监听下位机发送的图片信息，每发送完一张图片之后，将会等待上位机发 送来的接收成功或失败信息，若接收成功，则继续执行，否则断开连接。

```c
void Send_img(QString filepath)
{
    connect(); // 连 接 服 务 器
    Is_same(); // 动 态 监 控
    take_photo() // 拍 照
    receive_msg(); // 等 待 接 收 上 位 机 返 回 信 息
    return;
}
```

#### 4.2.2 传输阻值

阻值上传比图片简单，由于 TCP 传输的时字符串流，所以传输前需要将整形转化为字符串， 并发送即可。其中转化如下：

```c
char *int_to_string(int num)
{
    int i=0,j=0;
    char temp[10];
    while(num)
    {
    temp[i++]=num%10+'0'; //将 数 字 加 字 符0就 变 成 相 应 字 符
    num/=10; //此 时 的 字 符 串 为 逆 序
    }
    temp[i]='\0';
    i=i-1;
    while(i>=0)
    str[j++]=temp[i--]; //将 逆 序 的 字 符 串 转 为 正 序
    str[j]='\0'; //字 符 串 结 束 标 志
    return str;
}
```



## 5. 问题及解决办法

本次工作主要负责下位机与通信部分，在配置环境和书写代码方面遇到的问题还不少， 有如下几点。

- **问题一**

虚拟网络配置问题，本实验我们要求虚拟机和开发板能够互相传输数据，但是一开始 开发板和虚拟机不处于同一网段下，导致难以传输。这里可以单独在虚拟机网络下使 用管理员权限在 IPV4 设置中手动配置一个和开发板处于同一网段的 IP 地址即可， 但是要注意分配了地址后，很有可能在虚拟机下终端可能无法上网，也就是无法使用 install 等指令，需要重新配置回去才行。

- **问题二**

qmake 环境配置中，下载的 QT-4.7.1 的包，并配置好环境后，相利用 qmake 编译 QT 工程，这时候发现仅有在 qmake 目录下才能编译，要想在其他目录下编译时，需要打 开 qt 目录下 qmake.conf，将前三个 include 去掉一个“../”就可以在其他文件夹目录 下编译了，但注意这时候 qmake 的同时可能会自动帮你 make。

- **问题三**

Qt 兼容性问题，开发板兼容的是 QT-4.7.1，在 linux 系统下，默认安装的通常是 QT9 5，如果直接使用 QT-5 对开发板的项目进行书写，可能会报错。解决办法是，可以在 windows 下安装 qt4，用于开发板项目，而 linux 下 qt5 可用于上位机开发。

- **问题四**

摄像头刷新速率很低，所以导致后期动态检测时，延迟较大，效果不好，解决办法是 可以直接使用 usb 摄像头代替原本开发板的 OV5640.

- **问题五**

数据传输上，若客户端没有写等待服务器开启机制，并在服务器没开前，先开启了客 户端此时下位机的 qt 界面有可能会卡死，解决办法是最好先打开服务器，再打开客户 端。

