# PVE



## Network

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



## VM直通硬盘

```bash
# 查看硬盘
lshw -class disk -class storage
# 查看硬盘ID
ls -l /dev/disk/by-id/
# 直通硬盘，其中105为VM的ID，scsi1为VM中的硬盘设备号，后面为host中的硬盘设备
qm set 105 -scsi1 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC
```



https://pve.proxmox.com/wiki/Passthrough_Physical_Disk_to_Virtual_Machine_(VM)