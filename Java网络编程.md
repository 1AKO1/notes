# 网络编程

## 1.1 概述

### 计算机网络

计算机网络是指将==地理位置==不同的具有==多台计算机及其外部设备，通过通信线路连接起来==，在网络操作系统，网络管理软件及==网络通信协议==的管理和协调下，==实现资源共享==和信息传递的计算机系统（from 百度百科）



### 网络编程目的

- 数据交换
- 通信



### 想达到上述目的需要什么

1. 如何定位网络上的一台主机
   -  192.168.13.123：端口 
   - 通过IP地址 + 端口号 就可以定位到一台计算机的某个资资源
2. 如果找到了这个主机，如何传输数据？
   - 



## 1.2 网络通信的要素

### 如何实现网络的通信

- 通信双方地址
  - IP
  - 端口号
- 规则：网络通信协议
  - TCP/IP参考模型

![1590371738782](C:\Users\LJW20\AppData\Roaming\Typora\typora-user-images\1590371738782.png)

+ 小结
  + 网络编程中有两个问题
    + 如何准确的定位到网络上的一台或者多台主机
    + 找到主机之后如何通信
  + 网络通信中的要素
    + IP和端口号
    + 网络通信协议 ： TCP、 UDP
  + 万物皆对象
    + Java中必然有代表上述事物的对象，我们要操作对象完成目的



## 1.3 IP

InetAddress 对象

```java
InetAddress inetAddress1 = InetAddress.getByName("ip或者域名都可"); //确定主机名称的IP地址
InetAddress inetAddress2 = InetAddress.InetLocalHost(); //返回本地主机的地址
String ip = inetAddress2.getHostAddress(); //获取ip
String name = inetAddress2.getHostName(); //获取域名或者自己电脑的名字。
```

- 唯一定位一台网络上的计算机

- 127.0.0.1： 本机IP localhost

- ip地址分类

  - ipv4/ipv6

    - ipv4 ：127.0.0.1 四个字节组成， 0~255， 共有42亿个， 其中30亿都在北美，亚洲只有4亿， 2001年用尽

    - ipv6：128位， 8个无符号整数

      ```
      2001:abca:5ac6:005a:89ec:4664:0001:1316
      ```

  - 公网（互联网） -- 私网（局域网）

    - ABCD类地址
    - 192.168.xx.xx 是专门给组织内部使用





## 1.4 端口 port

端口表示计算机上的一个程序的进程

```java
InetSocketAddress socketAddress = new InetSocketAddress(hostname, port); //返回套接字地址
InetAddress inetAddress = socketAddress.getAddress();
String hostname = socketAddress.getHostName();
String port = socketAddress.getPort();
```

- 不同的进程有不同的端口号

- 端口号被规定为0~65535

- TCP和UDP都有0~65535； 所以可以跑 2 * 65535个进程

- 端口分类

  - 共有端口 0~1023

    - HTTP：80
    - HTTPS：443
    - FTP：21
    - Telent：23

  - 程序注册端口：1024~49151， 分配给用户或者程序的

    - Tomcat：8080
    - MySQL：3306
    - Oracle：1521

  - 动态、私有： 49152~65535

    ```bash
    netstate -ano # 查看所有的端口
    netstate -ano|findstr ”5900“ # 查看指定的端口
    tasklist|findstr ”8696“ #查看指定端口的进程
    Ctrl + shift + ESC // 打开任务管理器
    ```

    



## 1.5 通信协议

### 网络通信协议

在网络传输速率、传输码率、代码结构、传输控制等做了一系列的规定，很复杂。。

#### TCP/IP协议簇：实际上是一组协议

- 重要的协议
  - TCP：用户传输协议
  - UDP：用户数据报协议
- 出名的协议
  - TCP：传输控制协议
  - IP：网络互连协议



#### TCP UDP 对比

- TCP：打电话

  - 连接、稳定

  - `三次握手`，`四次挥手`

    ```
    最少需要三次，保证稳定连接
    A：你瞅啥？
    B：瞅你咋地？
    A：干一场！
    
    四次挥手，断开连接
    A:我要断开了。
    B：我知道你要断开了？
    B：你断了吗？
    A：我断了。
    ```

  - 客户端、服务端

  - 传输完成，释放连接，效率低

- UDP：发短信

  - 不连接、不稳定
  - 客户端、服务端：没有明确的界限
  - 不管有没有准备好、都可以发给你
  - DDOS：可以疯狂给你发包，然后就一直占用端口



## 1.6 TCP