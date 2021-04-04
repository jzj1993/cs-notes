# 网络相关知识



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

