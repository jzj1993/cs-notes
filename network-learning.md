# 网络相关知识



## OSI七层模型

![img](network-learning.assets/2018041112053246.png)

https://blog.csdn.net/qq_39521554/article/details/79894501



## 五层模型

https://www.ruanyifeng.com/blog/2012/05/internet_protocol_suite_part_i.html

![img](network-learning.assets/bg2012052902.png)



1. 物理层：电路连接，电气特性。
2. 数据链路层：广播通信。MAC地址。只有在一个子网才能广播通信。
3. 网络层：主机到主机通信。IP地址，子网掩码，路由。ARP协议（从IP地址得到MAC地址）。
4. 传输层：端口到端口通信。TCP / UDP
5. 应用层：HTTP / FTP / DHCP / DNS / ...



## 网络设备

Hub 集线器：一般工作在第一层。无源HUB直接连接所有端口；有源HUB会把收到的所有电信号放大并发送出去。智能Hub可能会工作在第二、三层。

> Hub只是一个多端口的信号放大设备，工作中当一个端口接收到数据信号时，由于信号在从源端口到Hub的传输过程中已有了衰减，所以Hub便将该信号进行整形放大，使被衰减的信号再生(恢复)到发送时的状态，紧接着转发到其他所有处于工作状态的端口上。
>
> https://www.163.com/dy/article/F2A2GO2E0511RN54.html

Switch 交换机：

- 第二层交换机：根据MAC地址转发数据。
- 第三层交换机：根据IP地址转发数据，具有部分路由器功能。交换机可以缓存IP和MAC地址对应关系，加速通信。
- 第四层交换机：根据IP和端口转发数据，可以提高服务器群的可靠性和可扩性。第三层交换机处理单一的包，但是第四层交换机可以识别端口信息，将特定业务的包发送给合适的计算机。

https://baike.baidu.com/item/%E7%AC%AC%E5%9B%9B%E5%B1%82%E4%BA%A4%E6%8D%A2

Router 路由器：一般工作在第三层。路由器可以把网络分割成逻辑上独立的网络单位，使网络具有一定的逻辑结构。路由器的主要工作就是为经过路由器的每个数据帧寻找一条最佳传输路径，并将该数据有效地传送到目的站点。

Bridge 网桥：简单理解成二层交换机，但是网桥由软件实现，交换机由硬件实现。

https://segmentfault.com/a/1190000022099473

AP，Access Point 无线接入点：有线信号转无线。

- Fit AP，瘦AP：通常说的AP指瘦AP。相当于无线交换机，不能独立工作，须配合AC集中管理。

- Fat AP，胖AP：自带操作系统，可以独立工作，需要单独配置。除了AP功能，还具备WAN、LAN端口，支持DHCP，PPPoE等。举例：常用的无线路由器。

https://zhuanlan.zhihu.com/p/64648479



## Gateway 网关

网关地址是一个网络通向其他网络的IP地址，网关通常就是路由器。



## 端口映射 / 虚拟服务器

将WAN口指定端口映射到LAN口指定设备的指定端口。



## DMZ主机

路由器的DMZ主机功能，可以把WAN口所有端口映射到指定LAN口设备的对应端口。



## UPNP

自动实现端口映射。

场景：

- 路由器WAN口有公网IP，LAN口接设备，路由器开启UPNP。

- 设备通过UPNP自动在路由器上配置端口转发，从而让公网IP某端口映射到设备的端口。
- 如果设备和公网之间有超过一层路由，UPNP无法正常工作。除非设置DMZ主机。



## DHCP服务器

DHCP服务器用于给新的设备分配IP地址。一个子网有一个DHCP服务器即可，有多个DHCP时可以设置其中一个为强制DHCP，覆盖其他DHCP服务器。

客户端连接DHCP服务器后，DHCP服务器可以下发IP、子网掩码、网关、DNS给客户端（网关和DNS默认为路由器自身IP）。

例如旁路由结构中，要让需要代理的客户端的网关和DNS指向旁路由，可以让主路由或者旁路由作为DHCP服务器下发设置。

https://www.right.com.cn/forum/thread-4035785-1-1.html

https://oeone.cn/archives/486.html



## DDNS

动态域名解析

实际案例：

- 家用网络公网IP一般不是静态地址，每次重新拨号都可能变化
- 想要稳定访问家里的设备，可以用OpenWRT配置DDNS服务
- 每次地址变化，将获取到的公网IP通过API更新到GoDaddy上的特定域名



## 家用路由器工作模式

- AP / Access Point / 访问点：上级路由器的有线LAN转无线。设备在同一个子网，IP由上级路由分配。网线接路由器的WAN和LAN都可以。
- Router / 无线路由：常规的无线路由器模式，WAN口网络进，LAN口和无线设备在新的子网。
- Repeater / 中继：放大Wifi信号，设备在新的子网，SSID相同（存疑）。
- Bridge / 桥接：放大Wifi信号，设备在同一个子网，SSID可以相同可以不同（存疑）。
- Client / 客户端：无线转有线，作为无线网卡使用。无线连Wifi，LAN口连需要上网的设备，设备在新的子网。
- Client Bridge / 客户端网桥：同Client，但设备在同一个子网。



https://zhuanlan.zhihu.com/p/32275116

https://www.tp-link.com/us/support/faq/442/

https://superuser.com/questions/410217/wireless-repeater-vs-wireless-bridge

https://www.zhihu.com/question/20380724/answer/83511160



## 光猫 路由模式 / 桥接模式

路由模式

```
       光纤
        |
  光猫拨号192.168.1.1
        |
主路由WAN口192.168.1.x
主路由LAN口192.168.5.1
        |
  设备192.168.5.x
```

桥接模式

```
光纤 - 光猫192.168.1.1 - 主路由拨号192.168.1.x
                                |
                          设备192.168.1.x
```



## 旁路网关 / 旁路由

```
       光猫
        |
主路由192.168.1.1 - OpenWRT单口旁路由192.168.1.2
        |
 设备192.168.1.x
```

主路由LAN口连接旁路由LAN口，旁路由下的设备在同一个子网。旁路由可提供网关、透明代理、DHCP、DNS等功能。

https://sspai.com/post/59708



## 光猫桥接 + 单臂路由拨号

```
光纤 - 光猫桥接 - 无线路由AP模式 - OpenWRT单臂路由拨号+DHCP
         |           |
      设备连接      电脑连接
```

- 光猫显示光纤EPON模式已连接

- 设备可以无线、有线连光猫，连接后可拨号上网
- IP：
  - 光猫LAN口192.168.1.1
  - 无线路由WAN / LAN口192.168.1.18
  - OpenWRT单臂路由WAN口公网IP，LAN口192.168.5.2

- 当电脑网关改成192.168.1.1，可以访问光猫和无线路由

- 当电脑网关用DHCP默认分配的192.168.5.2，可以上网，不能访问光猫和无线路由

理解（是否准确？）：

- 光猫 - 无线路由 组成 VLAN1（192.168.1.x）
- 无线路由 - OpenWRT - 电脑 组成VLAN2（192.168.5.x）

问题：为什么电脑在VLAN2不能访问光猫和无线路由？因为VLAN2被隔离了？



## Docker的网络模型

Docker的四种网络模型

- bridge：默认。创建虚拟网桥docker0，container连接到这个网桥上，处于独立的子网中。相当于接了一个路由器。外网不能直接访问容器，需要在docker创建容器做端口绑定（`docker run -p 8080:80 xxx`）。
- host：直接使用host的IP和端口，和host共享network namespace（`docker run --network host xxx`）。
- none：创建单独的network namespace，不配置网卡、IP等信息。
- container：和其他容器共享network namespace。

http://shareinto.github.io/2017/07/10/docker-ip/

https://stackoverflow.com/questions/30677702/trouble-running-upnp-on-docker

