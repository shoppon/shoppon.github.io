---
title: "在Openstack平台使用Windows 10"
categories: ["云计算"]
tags: ["openstack", "windows10"]
date: 2020-08-31T00:33:25+08:00
---

# KVM镜像制作

创建空的镜像文件

`qemu-img create -f qcow2 /home/shoppon/images/windows-10.qcow2 160G`

使用ISO创建镜像

`virt-install --connect qemu:///system  --name windows-10 --ram 8192 --vcpus 8  --network network=default,model=virtio  --disk path=/home/shoppon/images/windows-10.qcow2,format=qcow2,device=disk,bus=virtio  --cdrom /home/shoppon/images/win10_1909.iso  --disk path=/home/shoppon/images/virtio-win-0.1.171.iso,device=cdrom  --vnc --os-type windows`

将镜像转换为qcow2格式

`qemu-img convert -c -O qcow2 /home/shoppon/images/windows-10.qcow2 /home/shoppon/images/win10-openstack.qcow2`

创建glance镜像

`glance image-create --name "windows10" --file /tmp/windows-openstack.qcow2 --disk-format qcow2 --container-format bare --visibility public`

# BugFix

## 远程桌面黑屏
显卡透传配置成功后可以在`设备管理`中看到PCI设备，使用驱动精灵安装显卡驱动后即可启用显卡设备。开启远程桌面服务后无法使用MBP上的RDP软件进行连接，连接成功后显示黑屏。但是可以通过openstack中的VNC进行登陆。为了解决这个问题，需要将WDDM图形显示驱动禁用，具体操作步骤为：
- 在搜索栏中输入“编辑组策略”（gpedit.msc)，打开注册表修改窗口。
- 选择“计算机配置”-->“管理模板”-->“windows组件”-->“远程桌面服务”-->“远程桌面会话主机”-->“远程会话环境”目录。
- 将列表中的“为远程桌面程序使用WDDM图形显示驱动”选项禁用。


# 参考

- [远程桌面无法连接](https://answers.microsoft.com/en-us/windows/forum/all/remote-desktop-black-screen/63c665b5-883d-45e7-9356-f6135f89837a?page=2)