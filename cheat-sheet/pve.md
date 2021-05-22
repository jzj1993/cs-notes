# PVE



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

# restart network card
ifreload NETWORK
# or
ifdown NETWORK && ifup NETWORK
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



## Enterprise源

替换默认需要subscription的源为不需要subscription的源

```bash
vim /etc/apt/sources.list.d/pve-enterprise.list
```

```bash
deb http://download.proxmox.com/debian/pve stretch pve-no-subscription
# deb https://enterprise.proxmox.com/debian/pve buster pve-enterprise
```

也可以手动下载包

http://enterprise.proxmox.com/debian/pve/dists/buster/pvetest/binary-amd64/



## 安装ifupdown2

在网页修改网络参数，Apply Configuration的时候报错如下

> you need ifupdown2 to reload network configuration (500)

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