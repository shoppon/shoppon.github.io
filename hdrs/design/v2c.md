# V2C

## 背景

虚拟机启动顺序依次为**BIOS->MBR->GRUB->Kernal->Init->Runlevel**共6个步骤。

VMware虚拟机与云上虚拟机启动方式不一样，需要做适配才能在云上拉起。

## 方案

### 磁盘驱动

云上磁盘需要安装`virtio`驱动。

### initrd

`initrd`是一个临时文件系统，在启动阶段被内核调用，是启动所需要的目录和可执行程序的最小集合（包括`insmod`、`nash`等）。VMWare平台的initrd可能无法在引导华为云的虚拟化内核启动，因此需要根据云上虚拟化架构重建Linux `initrd`文件。制作开机初始化使用的`ramdisk`方式如下：

```shell
mkinitrd -f -v --with=mptsas --with=scsi_transport_sas --with=mptspi --with=mptscsih --with=mptbase --with=scsi_transport_spi --with=binfmt_misc --with=virtio_scsi --with=iscsi_boot_sysfs --with=virtio_blk --with=virtio_pci --with=iomirror /boot/initramfs-$(uname -r).img $(uname -r)
```

在配置文件` /etc/dracut.conf`中添加：

```shell
add_drivers+=" iomirror "
add_drivers+=" mptsas "
add_drivers+=" scsi_transport_sas "
add_drivers+=" mptspi "
add_drivers+=" mptscsih "
add_drivers+=" mptbase "
add_drivers+=" scsi_transport_spi "
add_drivers+=" binfmt_misc "
add_drivers+=" virtio_scsi "
add_drivers+=" iscsi_boot_sysfs "
add_drivers+=" virtio_blk "
add_drivers+=" virtio_pci "
```

然后执行`dracut -f`覆盖原来的配置。

### grub

[grub](https://zh.wikipedia.org/wiki/GNU_GRUB)是GNU规范的启动引导程序，它在启动时加载配置信息，并允许在启动时修改，如选择不同的内核和[initrd](https://zh.wikipedia.org/wiki/Initrd)。计算机启动后，[BIOS](https://zh.wikipedia.org/wiki/BIOS)将寻找第一个可启动的设备（通常为硬盘），而后从[MBR](https://zh.wikipedia.org/wiki/MBR)中加载启动程序，然后把控制交给这段代码。MBR位于硬盘的前512字节内。

修改grub配置文件`/etc/default/grub`（不同操作系统路径可能不一样，比如`/boot/grub/menu.lst`），去掉`GRUB_CMDLINE_LINUX`中硬编码问题，然后执行` grub2-mkconfig -o /boot/grub/grub.cfg`更新grub配置文件。

### fstab

`fstab`是磁盘挂载配置文件，如果使用盘符来挂载，容灾到云上之后盘符可能会发生变化，因此需要使用`blkid`来挂载：

```shell
#
# /etc/fstab
# Created by anaconda on Wed Aug 21 02:54:55 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=f8189f24-e06b-4bf0-9351-d930bb64cad7 /     ext4    defaults     1 1
```
