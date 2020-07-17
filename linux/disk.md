# 磁盘相关

## 概念

### 磁盘

- SCSI、SATA、U盘、Flash类型的磁盘：/dev/sdx，最多11（5到15）？
- IDE类型的磁盘：/dev/hdx，最多59（5到63）？
- VHD类型的磁盘：/dev/vdx，最多20？
- 虚拟化XEN：/dev/xvdx

### 扇区

最小的物理储存单元，每个扇区512Bytes。每一个扇区中包括**主启动区**（Master boot record, **MBR**, 446 Bytes）和**分割表**（partition table，64 Bytes）。还有2 Bytes呢？

### 分区

对磁盘进行分割，以格式/dev/hdxn(/dev/hda1)进行编号，日志分区、用户分区

主分区：每个硬盘设备最多4个主分区，用来进行操作系统启动

扩展分区

逻辑分区

### 文件系统

对分区进行格式化，使操作系统能够识别

### iNode

包含文件权限（rwx）和文件属性（拥有者、群组、时间）属性。一个文件一个inode

使用`ls -il`可查看文件的inode号。

### Superblock

记录文件系统的整个信息（格式、使用量、剩余量），一般为1024 Bytes，可使用`dumpe2fs`命令来观察superblock信息

### block

实际记录文件的内容，一个文件可使用多个block，一个block只能给一个文件使用，一个block可以设置1K、2K、4K。

## LVM

**创建PV：**`pvcreate /dev/xvde`

**创建VG：**`vgcreate vg1 /dev/xvde`

**添加新的物理卷到VG中：**`vgextend vg1 /dev/xvdf`

**创建LV：**`lvcreate -l 5120 -n oma vg1`

**创建文件系统：**`mkfs.ext4 -j /dev/vg1/oma`

**挂载文件系统：**`mount /dev/vg1/oma /home/oma/`

 **开机自启动：**`echo "/dev/vg1/oma /home/oma/ ext4 defaults 0 0" >> /etc/fstab`

**查询磁盘ID：**`blkid`

**查询块设备信息及其结构：**`lsblk`

**重启后重新激活lv：**`lvchange -ay vg1/thinpool`

**扩容后让lv大小生效：**`resize2fs /dev/mapper/nfs-ovirt`

## FIO

顺序写

```shell
fio -filename=/dev/sda -direct=1 -iodepth 1 -thread -rw=write -ioengine=psync -bs=16k -size=1G -numjobs=4 -runtime=1000 -group_reporting -name=sqe_100write_4k
```

随机写

```shell
fio -filename=/dev/vdb -direct=1 -iodepth 1 -thread -rw=randwrite -ioengine=psync -bs=4k -size=1G -numjobs=4 -runtime=180 -group_reporting -name=rand_100write_4k
```

### dd

使用`dd`写文件：`dd if=/dev/urandom bs=8192 count=125000 of=/root/1Gb.file`

### hexdump

**查看指定设备特定位置数据：**`hexdump -n 1024 -x -s 1G /dev/vdb`

### iostat

**每隔1秒查看磁盘速率：**`iostat -m 1`

### 刷盘

**将系统缓存刷盘：**`sync`

**丢弃内核clean cache：**`echo 3 | sudo tee /proc/sys/vm/drop_caches`

**调用ioctl刷新缓存：**`blockdev --flushbufs /dev/sda`

**Flush the on-drive write cache buffer：** `hdparm -F /dev/sda`

## 参考

- [IO测试工具之fio详解](https://www.cnblogs.com/raykuan/p/6914748.html)
