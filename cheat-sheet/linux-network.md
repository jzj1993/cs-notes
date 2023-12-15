# Linux 网络相关

[TOC]

## 相关工具

- ifupdown
- netplan
- NetworkManager
- net-tools: netstat, ifconfig (deprecated)...
- ip
- traceroute
- telnet
- ping
- nc



```bash
# 测试IP和域名(ICMP协议)
ping 8.8.8.8
ping www.google.com

# 查看DNS解析结果。如果设置了代理，结果会指向代理服务器。
nslookup www.google.com

# 测试网址
curl www.google.com

# 查看访问指定网址经过的设备和IP地址
traceroute www.google.com

# 查看网络连接
netstat -anp

# 测试指定端口TCP连接
telnet localhost 8888

# 测试指定端口TCP/UDP连接，参数v指定显示verbose信息，参数z指定仅测试。
nc -vz  192.168.1.1 53  # TCP
nc -vuz 192.168.1.1 53  # UDP

# 监听指定端口并显示接收到的信息
nc -l 8888

# 连接到端口并发送信息。连接成功后再一端输入字母并按回车，在另一端就可以输出显示。
nc localhost 8888
```



## ip命令

```bash
# 查看网卡和IP
ip a
ip addr
ip address

# 查看固定IP
ip address show permanent

# 监听ip变化事件
ip monitor

# 查看网卡信息
ip link

# 查看网关
ip route show default
# 查看IPv6网关
ip -6 route show default
```

[routing - Where is this IPv6 address coming from? - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/352544/where-is-this-ipv6-address-coming-from)

[ip-address: protocol address management - Linux Man Pages (8) (systutorials.com)](https://www.systutorials.com/docs/linux/man/8-ip-address/)



## ifupdown / ifupdown2 工具

[User Guide — ifupdown2 0.1 documentation (cumulusnetworks.github.io)](https://cumulusnetworks.github.io/ifupdown2/ifupdown2/userguide.html)

[Ubuntu Manpage: ifreload - reload network interface configuration](http://manpages.ubuntu.com/manpages/xenial/man8/ifreload.8.html)

PVE和部分较早版本Ubuntu使用了ifupdown工具管理网络

配置

```bash
vim /etc/network/interfaces

# restart network card
ifreload eth0
# or
ifdown eth0 && ifup eth0
```



## netplan (Ubuntu 18.04+)

Ubuntu 18.04+使用netplan管理网络

确保禁用cloud init

```bash
# disable cloud init
sudo vim /etc/cloud/cloud.cfg.d/subiquity-disable-cloudinit-networking.cfg
# modify to this if it is not:
# network: {config: disabled}
```

配置修改和应用

```bash
# modify network config file (ubuntu 20.04)
sudo vim /etc/netplan/00-installer-config.yaml

# try config with auto rollback
sudo netplan try
# apply config permanently
sudo netplan apply
# reboot
sudo reboot
```

配置文件示例

1、指定使用NetworkManager (Ubuntu Desktop默认配置)

```bash
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
```

2、静态IP

```bash
network:
  version: 2
  ethernets:
    ens18:
      addresses: [192.168.5.20/24]
      gateway4: 192.168.5.2
      nameservers:
        addresses: [114.114.114.114, 223.5.5.5]
```

3、DHCP动态下发IP

```bash
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: true
```

https://linuxhint.com/setup_static_ip_address_ubuntu/

https://ubuntu.com/server/docs/network-configuration



## OpenWRT Network

OpenWRT使用自己的网络管理工具

https://openwrt.org/zh-cn/doc/uci/network

```bash
# 修改网卡参数
vim /etc/config/network
# 重启网卡应用配置
/etc/init.d/network restart
```

示例配置文件

```bash
config interface 'loopback'
	option ifname 'lo'
	option proto 'static'
	option ipaddr '127.0.0.1'
	option netmask '255.0.0.0'

config globals 'globals'
	option ula_prefix 'fdbd:9c99:7971::/48'
	option multipath 'disable'
	option mptcp_path_manager 'fullmesh'
	option mptcp_scheduler 'default'
	option mptcp_checksum '0'
	option mptcp_debug '0'
	option mptcp_syn_retries '5'
	option mptcp_fullmesh_num_subflows '1'
	option mptcp_fullmesh_create_on_err '1'
	option mptcp_ndiffports_num_subflows '1'
	option congestion 'olia'

config interface 'lan'
	option type 'bridge'
	option ifname 'eth0'
	option proto 'static'
	option ipaddr '192.168.199.200'
	option netmask '255.255.255.0'
	option ip6assign '60'
	option multipath 'off'
	option gateway '192.168.199.1'
	option dns '192.168.199.1'

config interface 'wan'
	option ifname 'eth1'
	option proto 'dhcp'
	option auto '0'

config interface 'wan6'
	option ifname 'eth1'
	option proto 'dhcpv6'
	option auto '0'
```



## 命令行设置HTTP(S)代理

仅对部分会主动读取代理环境变量的软件有效，例如curl/wget。不同软件使用的格式可能不一样，全都设置上了。

```bash
_HTTP_PROXY=http://127.0.0.1:1087

setproxy() {
    export http_proxy=$_HTTP_PROXY;
    export https_proxy=$_HTTP_PROXY;
    export all_proxy=$_HTTP_PROXY;
    export HTTP_PROXY=$_HTTP_PROXY;
    export HTTPS_PROXY=$_HTTP_PROXY;
    export ALL_PROXY=$_HTTP_PROXY;
    echo "set http(s) proxy = $_HTTP_PROXY";
}

unsetproxy() {
    unset http_proxy;
    unset https_proxy;
    unset all_proxy;
    unset HTTP_PROXY;
    unset HTTPS_PROXY;
    unset ALL_PROXY;
    echo "unset http(s) proxy";
}
```



## git设置代理

git url 是 HTTP 形式：使用 `git config` 配置

```bash
git config --global http.proxy "http://127.0.0.1:8080"
git config --global https.proxy "http://127.0.0.1:8080"
# or
git config --global http.proxy "socks5://127.0.0.1:1080"
git config --global https.proxy "socks5://127.0.0.1:1080"
```

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

git url 是 SSH 形式：修改 `~/.ssh/config` 文件（不存在则新建）

```bash
Host github.com
   HostName github.com
   User git
   # 以下两者选一个
   # 1. 走 HTTP 代理
   # ProxyCommand socat - PROXY:127.0.0.1:%h:%p,proxyport=8080
   # 2. 走 socks5 代理（如 Shadowsocks）
   # ProxyCommand nc -v -x 127.0.0.1:1080 %h %p
```

参考 https://gist.github.com/chuyik/02d0d37a49edc162546441092efae6a1



## 软件防火墙 ufw

https://www.linode.com/docs/guides/configure-firewall-with-ufw/

