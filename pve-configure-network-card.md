# PVE换网卡后重新设置网络

## 背景

由于换了主机，直接把原有的SSD放到新的设备上了，PVE能正常启动但是不能联网。

在网上查了一通发现是因为网卡问题，网卡变了需要自己修改网卡的名字。

```bash
# 查看网卡设备，有线网卡一般是enXxx
ip link show

# 修改配置文件，把网卡改成新的名字
nano /etc/network/interfaces

# 重启网络应用改动
/etc/init.d/networking restart
```

示例配置文件如下，`ens33` 为原先的有线网卡，需要更新。

```bash
auto lo
iface lo inet loopback

iface ens33 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.5.11/24
        gateway 192.168.5.2
        bridge_ports ens33
        bridge_stp off
        bridge_fd 0
```

https://forum.proxmox.com/threads/how-to-reset-networking-after-replacing-nic.38178/



## 查看网卡

查看网卡，发现除了环回网卡 `lo` 和桥接虚拟网卡 `vmbr0`，只有一个无线网卡 `wlo1`，而有线网卡没有被识别。我的主板是MAG B550M MORTAR WIFI，有线网卡型号 Realtek® RTL8125B 2.5G LAN。因为网卡太新了，使用的PVE Linux内核版本不支持，需要装驱动。

```bash
# 查看网卡设备
ip link show

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlo1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 3c:9c:0f:8e:93:42 brd ff:ff:ff:ff:ff:ff
3: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 00:e0:4c:53:44:58 brd ff:ff:ff:ff:ff:ff
```

https://unix.stackexchange.com/questions/134483/why-is-my-ethernet-interface-called-enp0s10-instead-of-eth0



## 编译安装驱动

为了方便，插上了一个比较老的USB网卡，修改了network配置连上了PVE。

从Realtek官网下载到Linux驱动 2.5G Ethernet LINUX driver r8125 for kernel up to 5.6

https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software



解压后发现需要运行 `autorun.sh` 编译。

编译报错找不到make和gcc于是 `apt install make gcc` 安装。还遇到报错：

```
Check old driver and unload it.
Build the module and install
make[2]: *** /lib/modules/5.4.73-1-pve/build: No such file or directory.  Stop.
make[1]: *** [Makefile:171: clean] Error 2
make: *** [Makefile:48: clean] Error 2
```



网上搜了一下发现是缺少kernel headers，也就是内核的头文件

https://www.reddit.com/r/debian/comments/c5uyn5/cannot_make_anything_due_to_missing_file_or/



用 `uname -r`  查看内核版本是 `5.4.73-1-pve` 。按照网上的办法直接跑 `apt install linux-headers-$(uname -r)` 是找不到对应的headers的，因为PVE有自己的headers。需要从PVE的源下载安装，即 `apt install pve-headers-VERSION`。

https://forum.proxmox.com/threads/linux-headers-5-0-18-1-pve.57237/



但是由于我用的不是enterprise版本，这个源也不能直接用，于是手动去下载了对应的deb文件，然后 `dpkg -i XXX.deb` 安装。

http://enterprise.proxmox.com/debian/pve/dists/buster/pvetest/binary-amd64/



然后再次运行 `autorun.sh` 编译，虽然还有报错但是最后完成了。

```bash
Check old driver and unload it.
Build the module and install
Warning: modules_install: missing 'System.map' file. Skipping depmod.
DEPMOD 5.4.73-1-pve
load module r8125
Updating initramfs. Please wait.
update-initramfs: Generating /boot/initrd.img-5.4.73-1-pve
Running hook script 'zz-pve-efiboot'..
Re-executing '/etc/kernel/postinst.d/zz-pve-efiboot' in new private mount namespace..
No /etc/kernel/pve-efiboot-uuids found, skipping ESP sync.
Completed.
```



## 重新配置网络

再次查看发现网卡 `enp42s0` 已经正确识别了。

```bash
ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: wlo1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 3c:9c:0f:8e:93:42 brd ff:ff:ff:ff:ff:ff
4: enx00e04c534458: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master vmbr0 state UP mode DEFAULT group default qlen 1000
    link/ether 00:e0:4c:53:44:58 brd ff:ff:ff:ff:ff:ff
5: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 00:e0:4c:53:44:58 brd ff:ff:ff:ff:ff:ff
6: enp42s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 2c:f0:5d:ac:66:36 brd ff:ff:ff:ff:ff:ff
```



再次修改network配置使用这个新的网卡即可。