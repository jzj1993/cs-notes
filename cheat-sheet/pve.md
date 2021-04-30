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
ifdown NETWORK
ifup NETWORK
```





## 下载enterprise包

默认只有enterprise才能直接从apt源安装pve的包，可以自己手动在这里下载

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