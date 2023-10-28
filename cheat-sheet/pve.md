# PVE常用配置

## 网络基础配置

```bash
# show ip address
ip addr

# modify network settings
nano /etc/network/interfaces
nano /etc/issue
nano /etc/hosts

# restart network
/etc/init.d/networking restart
# or
systemctl restart networking

# restart network card
ifreload NETWORK
# or
ifdown NETWORK && ifup NETWORK
```

静态IP示例：

```bash
auto lo
iface lo inet loopback

iface enp1s0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.124.17/24
        gateway 192.168.124.1
        bridge-ports enp1s0
        bridge-stp off
        bridge-fd 0
```

DHCP动态获取IP示例：

```bash
auto lo
iface lo inet loopback

iface enp1s0 inet manual

auto vmbr0
iface vmbr0 inet dhcp
        bridge-ports enp1s0
        bridge-stp off
        bridge-fd 0
```


修改PVE启动时显示的欢迎文字中的IP地址

```bash
nano /etc/issue
nano /etc/hosts
```

```bash

------------------------------------------------------------------------------

Welcome to the Proxmox Virtual Environment. Please use your web browser to 
configure this server - connect to:

  https://192.168.5.12:8006/

------------------------------------------------------------------------------

```

## 固定IPv6配置

如果希望PVE Host或LXC有IPv6，则需要给Host配置IPv6。可以在网页界面或者命令行中配置固定IP。此方式仅适用于路由器IPv6固定不变的场景。

IPv6+网关地址的确定：

- 找到路由器的Linked IP Address作为网关，例如 `fe80::xx:xx:xx:dd3`。
- 找到路由器的IPv6地址，例如 `240e:xx:xx:b6c1::1001/64`，前面4组64位作为IPv6前缀。
- 找到网桥vmbr0的mac地址，使用 [EUI-64计算器](https://eui64-calc.princelle.org/) 计算出后4组64位地址，结合上个步骤中的前缀，即可组成完整的IPv6，例如 `240e:xx:xx:b6c1:1ac0:xx:xx:9818`。



## aliyun源

```bash
vim /etc/apt/sources.list
```

其中`buster`根据PVE版本不同
```bash
deb http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb http://mirrors.aliyun.com/debian-security buster/updates main
deb-src http://mirrors.aliyun.com/debian-security buster/updates main
deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
```

PVE 8.0默认的源文件是
```bash
deb http://ftp.debian.org/debian bookworm main contrib

deb http://ftp.debian.org/debian bookworm-updates main contrib

# security updates
deb http://security.debian.org bookworm-security main contrib
```

## Enterprise源

默认的Enterprise源在 apt update时会报错

```
E: Failed to fetch https://enterprise.proxmox.com/debian/ceph-quincy/dists/bookworm/InRelease  401  Unauthorized [IP: 51.79.228.122 443]
E: The repository 'https://enterprise.proxmox.com/debian/ceph-quincy bookworm InRelease' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
```

替换默认需要subscription的源为不需要subscription的源

```bash
nano /etc/apt/sources.list.d/pve-enterprise.list
```

其中`buster`根据PVE版本不同
```bash
deb http://download.proxmox.com/debian/pve buster pve-no-subscription
# deb https://enterprise.proxmox.com/debian/pve buster pve-enterprise
```

```
nano /etc/apt/sources.list.d/ceph.list
```

```
deb http://download.proxmox.com/debian/ceph-quincy bookworm enterprise
#deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
```

也可以手动下载包

http://enterprise.proxmox.com/debian/pve/dists/buster/pvetest/binary-amd64/


## 去掉网页登录后No Valid Subscription弹窗

参考： https://johnscs.com/remove-proxmox51-subscription-notice/

```bash
cd /usr/share/javascript/proxmox-widget-toolkit
cp proxmoxlib.js proxmoxlib.js.bak
vim proxmoxlib.js
# remove Ext.Msg.show({title: gettext('No valid subscription')
systemctl restart pveproxy.service
```

## 安装ifupdown2

在网页修改网络参数，Apply Configuration的时候报错如下

```
you need ifupdown2 to reload network configuration (500)
```

安装 `ifupdown2` 即可解决。首先确保已经添加了 `pve-no-subsciption` 源，然后用apt安装即可：

```bash
apt update && apt install -y ifupdown2
```

[How to fix the "Apply Configuration" button | Proxmox Support Forum](https://forum.proxmox.com/threads/how-to-fix-the-apply-configuration-button.73053/)



## PCI(e)设备直通

PCI设备直通后，虚拟机可以直接访问PCI硬件，包括网卡、NVMe固态硬盘、SATA控制器、显卡等。

[Pci passthrough - Proxmox VE](https://pve.proxmox.com/wiki/Pci_passthrough)



## 硬盘设备直通

如果电脑上有多个SATA控制器硬件，可以直接用PCI直通的方式，让虚拟机直通SATA控制器。如果只有一个SATA控制器且PVE本身也占用了，就只能设置硬盘设备直通。

```bash
# 查看硬盘
lshw -class disk -class storage
# 查看硬盘ID
ls -l /dev/disk/by-id/
# 直通硬盘，其中105为VM的ID，scsi1为VM中的硬盘设备号，后面为host中的硬盘设备
qm set 105 -scsi1 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC
```

[Passthrough Physical Disk to Virtual Machine (VM) - Proxmox VE](https://pve.proxmox.com/wiki/Passthrough_Physical_Disk_to_Virtual_Machine_(VM))



## 开启硬件嵌套虚拟化

开启硬件嵌套虚拟化后，可以在虚拟机里嵌套虚拟化时大幅提升性能，例如PVE装Windows，Windows里嵌套使用WSL2。

[Nested Virtualization - Proxmox VE](https://pve.proxmox.com/wiki/Nested_Virtualization)



## 安装 Win10 最佳实践

[Windows 10 guest best practices - Proxmox VE](https://pve.proxmox.com/wiki/Windows_10_guest_best_practices)
