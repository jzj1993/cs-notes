# Linux磁盘操作



磁盘可以从三个层级来看

1. 磁盘硬件设备
2. 磁盘分区
3. 文件系统



## 磁盘设备、分区查看 df lsblk fdisk



```bash
$ df -h               # 查看挂载的磁盘分区（文件系统），不支持LVM
Filesystem      Size  Used Avail Use% Mounted on
# ...
udev            3.9G     0  3.9G   0% /dev
/dev/sda2        63G  6.3G   54G  11% /
/dev/nvme0n1p1  229G  185G   32G  86% /data


$ lsblk               # 查看磁盘和分区（磁盘）
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
# ...
sda           8:0    0    64G  0 disk
├─sda1        8:1    0     1M  0 part
└─sda2        8:2    0    64G  0 part /
sr0          11:0    1   1.1G  0 rom
nvme0n1     259:0    0 232.9G  0 disk
└─nvme0n1p1 259:1    0 232.9G  0 part /data


$ sudo fdisk -l       # 查看磁盘设备（设备）
# ...
Disk /dev/nvme0n1: 232.91 GiB, 250059350016 bytes, 488397168 sectors
Disk model: Samsung SSD 970 EVO 250GB
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xf45f1fdb

Device         Boot Start       End   Sectors   Size Id Type
/dev/nvme0n1p1 *     2048 488396799 488394752 232.9G 83 Linux


Disk /dev/sda: 64 GiB, 68719476736 bytes, 134217728 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: B328CE9B-8122-48B7-830A-D46CDAFD0A87

Device     Start       End   Sectors Size Type
/dev/sda1   2048      4095      2048   1M BIOS boot
/dev/sda2   4096 134215679 134211584  64G Linux filesystem

$ ll /dev/disk/by-uuid    # 根据UUID查看磁盘
$ ll /dev/disk/by-id      # 根据ID查看磁盘
```

## 分区工具 cfdisk

```bash
# 打开硬盘
cfdisk /dev/sdc

# 交互界面编辑分区
```

## 分区工具 fdisk

```bash
# 列举硬盘
fdisk -l
# 进入分区
fdisk /dev/sdc
# m 帮助
# p 显示当前分区
# n 新建分区
# d 删除分区
# w 保存分区结果
```

分区对齐

https://askubuntu.com/questions/156994/partition-does-not-start-on-physical-sector-boundary



## 分区工具 parted

```bash
# 执行操作前先umount相关分区
sudo umount /dev/sdc1

# 启动parted并指定磁盘
sudo parted /dev/sdc

# 查看帮助
(parted) help

# 打印信息
(parted) print

# 分区大小调整
(parted) resizepart
Partition number? 1
End?  [107GB]? 215GB
# or
(parted) resizepart 1 100%

# 退出
(parted) quit
```



## 格式化分区（创建文件系统） mkfs

```bash
# 格式化分区为ext4
mkfs.ext4 /dev/sdc1
```



## 分区挂载 mount umount

```bash
sudo mount /dev/sdb1 /data  # 挂载磁盘分区到目录

# 配置开机自动挂载。注意，fstab配置错误可能导致无法开机。
sudo vim /etc/fstab

# 格式：分区 挂载目录 文件系统 defaults 0 0
# <file system> <mount point> <type> <options> <dump> <pass>
# 示例（建议用UUID因为比较固定）
# /dev/sdb1 /data ext4 defaults 0 0
# UUID=xxx /data ext4 defaults 0 0
# /dev/by-uuid/xxx /data/ext4 defaults 0 0

sudo mount -a # 应用fstab配置
sudo mount -l # 查看挂载信息
sudo umount /mnt/hdd # 取消挂载
```



解决 umount `target is busy` 问题

```bash
# 查看占用的PID
lsof /mnt/xxx

# 停止相关进程
kill PID
```



## 检查分区 e2fsck

检查和修复ext分区

```bash
sudo umount /dev/sdb
e2fsck /dev/sdb -fy
```


## LVM

进入lvm工具
```bash
lvm
```
LVM工具常用命令

```bash
# 查看帮助
lvm> help
```

列举volume
```bash
# 查看physical volume
lvm> pvs
  PV             VG  Fmt  Attr PSize    PFree 
  /dev/nvme0n1p3 pve lvm2 a--  <931.01g 15.99g

# 查看volume group
lvm> vgs
  VG  #PV #LV #SN Attr   VSize    VFree 
  pve   1  14   0 wz--n- <931.01g 15.99g

# 查看logical volume
lvm> lvs
  LV                                       VG  Attr       LSize    Pool Origin                            Data%  Meta%  Move Log Cpy%Sync Convert
  data                                     pve twi-aotz-- <794.79g                                        87.11  4.20                            
  root                                     pve -wi-ao----   96.00g                                                                               
  snap_vm-100-disk-0_EnvironmentConfigured pve Vri---tz-k   64.00g data                                                      
  swap                                     pve -wi-ao----    8.00g                                                                               
  vm-100-disk-0                            pve Vwi-a-tz--   64.00g data snap_vm-100-disk-0_GnomeInstalled 7.76                                   
  vm-100-state-EnvironmentConfigured       pve Vwi-a-tz--  <16.49g data                                   5.68                                   
  vm-101-disk-0                            pve Vwi-a-tz--   32.00g data                                   14.36                                  
  vm-103-disk-0                            pve Vwi-a-tz--   32.00g data                                   25.77     
```

查看volume具体信息
```bash
pvdisplay   # show physical volumes
vgdisplay   # show volume groups (including free space available)
lvdisplay   # show logical volumes
```

## LVM分区扩容/缩容

物理分区

```bash
# Enlarge the physical volume to occupy the whole available space in the partition
pvresize /dev/vda3
```

逻辑分区

```bash
# 查看LVM逻辑分区
lvdisplay

# 扩容到100G。其中 /dev/xxx 为 LV Path
lvextend -L 100G /dev/xxx

# 增加20G
lvresize --size +20G --resizefs /dev/xxx

# 扩容到最大
lvextend -l +100%FREE /dev/xxx

# 扩容到最大
lvresize --extents +100%FREE --resizefs /dev/xxx

# 缩容到80G
lvreduce -L 80G /dev/xxx
```



[lvm磁盘在线扩缩容 | Louis Blog (fenghong.tech)](https://www.fenghong.tech/blog/ops/lvm-reduce-extend/)

[Resize disks - Proxmox VE](https://pve.proxmox.com/wiki/Resize_disks#3._Enlarge_the_filesystem.28s.29_in_the_partitions_on_the_virtual_disk)



## Linux虚拟机磁盘扩容步骤

1. Host上扩容VM的磁盘（VPS后台操作，或PVE调整LVM分区大小等）
2. Guest中扩容分区
3. Guest中扩容文件系统

缩容步骤与上述相反。



### 使用parted扩容分区

```bash
sudo umount /dev/sdc1
sudo parted /dev/sdc

# print info
(parted) print

# use resizepart to resize partition
(parted) resizepart
Partition number? 1
End?  [107GB]? 215GB
# or
(parted) resizepart 1 100%

# quit parted
(parted) quit
```



### 使用resize2fs扩容文件系统（非LVM文件系统）

```bash
# verify partitio consistency
sudo e2fsck -f /dev/sdc1

# resize file system
sudo resize2fs /dev/sdc1
```



[Expand virtual hard disks on a Linux VM - Azure Virtual Machines | Microsoft Docs](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/expand-disks)

[Resize disks - Proxmox VE](https://pve.proxmox.com/wiki/Resize_disks)

