# Linux磁盘操作



## 磁盘和分区查看

```bash
$ df -h               # 挂载磁盘占用率
Filesystem      Size  Used Avail Use% Mounted on
# ...
udev            3.9G     0  3.9G   0% /dev
/dev/sda2        63G  6.3G   54G  11% /
/dev/nvme0n1p1  229G  185G   32G  86% /data


$ lsblk               # 查看磁盘和分区
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
# ...
sda           8:0    0    64G  0 disk
├─sda1        8:1    0     1M  0 part
└─sda2        8:2    0    64G  0 part /
sr0          11:0    1   1.1G  0 rom
nvme0n1     259:0    0 232.9G  0 disk
└─nvme0n1p1 259:1    0 232.9G  0 part /data


$ sudo fdisk -l       # 查看磁盘设备
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



## fdisk分区

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



## 格式化分区

```bash
# 格式化分区为ext4
mkfs.ext4 /dev/sdc1
```



## 磁盘挂载

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



