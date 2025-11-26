---
title: UDP&&TCP Note
---

## 1.ip ##

---

### 1.1 ip概念 ###    

**ip地址** ：互联网协议地址，用于唯一标识计算机    
**ip地址分类** ：公网ip和内网ip，ipv4和ipv6    
**特殊ip地址** ：127.0.0.1、localhost表示本机ip    
**ip常用命令** ：ipconfig 查看ip, ping 测试ip连通性

### 1.2 Java中获取ip ###    

**获取本机ip** ： ` public static InetAddress getLocalHost() `    
**获取指定ip** ： ` public static InetAddress getByName(String host) `    
**获取ip地址对象** ：

- *对应的主机名*： ` public String getHostName() `
- *应的地址* ： ` public String getHostAddress() `

**指定时间内能否连通** ： ` public boolean isReachable(int timeout) `

---

## 2.端口号 ##    

### 2.1 端口号概念 ###

**端口号** ：计算机网络中用于标识计算机进程的数字,范围0-65535
> 0-1023为特殊端口，1024-49151为用户端口，49152-65535为系统端口    
> 一个设备中应用程序不能使用相同的端口号。

---

## 3.协议  ##

## 3.1 UDP协议 ## 

**1.UDP**　：
- *特点*：用户数据报协议，无连接，数据不可靠，速度快；
- *包含数据*：源ip、源端口、目的ip、目的端口、数据（64KB内）
- *应用*：语音通话，视频直播...

**2.Java中相关代码** ：
- *创建客户端的socket对象（随机分配端口）* ： ` DatagramSocket socket = new DatagramSocket(); `
- *创建服务器的socket对象（指定自身端口）* ： ` DatagramSocket socket = new DatagramSocket(int port); `
- *创建数据包对象* ： ` DatagramPacket packet = new DatagramPacket(byte[] data, int data.length, InetAddress address, int port); ` 
- *发送数据包* ： ` socket.send(DatagramPacket packet); `
- *接收数据包* ： ` socket.receive(DatagramPacket packet); `
- *获取数据* ： ` byte[] data = packet.getData(); `
- *数据转为字节* ： ` byte[] bytes = 数据.getBytes(); `

>socket对象：Socket 是应用层与 TCP/IP 协议族之间的中间抽象层，它提供了一组标准化的接口，用于实现网络通信。    
>接收数据包：receive方法是阻塞方法，调用该方法会阻塞进程，直到收到数据包。

## 3.2 TCP协议 ##

**TCP**　：
- *特点*：传输控制协议，有连接，数据可靠，传输速度慢
- *三次握手*：SYN、SYN+ACK、ACK 建立连接
- *四次挥手*：FIN、FIN+ACK、ACK、RST 断开连接
- *应用*：网页、邮件传输、文件下载...
