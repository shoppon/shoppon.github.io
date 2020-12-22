---
typora-root-url: ../

---

# 概览

EasyStack作为中立开放的云计算厂商，支持对接各种第三方商业存储。

# 对接前准备

## 网络规划

ECS使用管理平面和杉岩存储的管理平面通信，发送登录、查看存储池、创建卷等REST请求。

ECS使用`storagepub`平台和杉岩存储的业务平台通信，实现卷的IO相关操作。

对接前需要保证网络已按要求配置完毕，ECS平台与杉岩存储网络已打通。

## 信息收集

对接前需要收集存储相关信息：

| 收集项                                | 样例                                         |
| ------------------------------------- | -------------------------------------------- |
| 存储管理网络IP地址                    | 192.168.4.6                                  |
| 存储管理员账号                        | admin                                        |
| 存储管理员密码                        | admin                                        |
| 为控制节点提供iSCSI目标器的IP地址列表 | 192.168.30.101,192.168.30.102,192.168.30.103 |
| chap认证用户名                        | chapuser                                     |
| chap认证密码                          | chappassword                                 |

信息收集后反馈给研发制作存储对接包。

## 平台配置

### 启动器配置

对接IPSAN时，`nova-compute`的POD中的启动器名称需要使用宿主机的配置，步骤如下：

1. SSH登陆ECS控制节点。

2. 运行`kubectl edit daemonsets.apps -n openstack nova-compute`，进入kubernetes daemonsets修改界面。

3. 在`volumeMounts`配置下新增

   ```shell
   -name: host-iscsi-dir
   	mountPath: /etc/iscsi/
   ```

   在`volumes`配置下新增

   ```shell
   -name: host-iscsi-dir
   	hostPath:
   	path: /etc/iscsi/
   	type: DirectoryOrCreate
   ```

### 更新repo源

对接前将IPSAN所需要的iscsi rpm包更新到roller的repo源，登陆控制节点依次执行以下命令：

```shell
export busy=$(k get po -n openstack|grep busybox|awk '{print $1}'|xargs -L 1);kubectl exec -it ${busy} -n openstack -- bash
kubectl exec -i -n openstack coaster-all-0 -c coaster-other -- mkdir -p /var/www/roller/centos/rollerweb/x86_64/Packages/updated_rpms
kubectl exec -i -n openstack coaster-all-0 -c coaster-other -- chmod 755 /var/www/roller/centos/rollerweb/x86_64/Packages/updated_rpms
kubectl cp iscsi-initiator-utils-6.2.0.874-4.el7.centos.es.x86_64.rpm -n openstack coaster-all-0:/var/www/roller/centos/rollerweb/x86_64/Packages/updated_rpms  -c coaster-other
kubectl cp iscsi-initiator-utils-iscsiuio-6.2.0.874-4.el7.centos.es.x86_64.rpm -n openstack coaster-all-0:/var/www/roller/centos/rollerweb/x86_64/Packages/updated_rpms  -c coaster-other
kubectl exec -i -n openstack coaster-all-0 -c coaster-other -- createrepo /var/www/roller/centos/rollerweb/x86_64 -g Packages/comps.xml
yum makecache
```

## 存储配置

对接前需要在存储上创建存储池和创建网关。

### 创建存储池

1. 登陆杉岩存储管理界面，从左侧菜单进入”资源管理“-->”逻辑池“界面。
2. 点击”创建“按钮，创建`vms`逻辑池；展开高级选项，勾选”第三方资源“。
3. 具体创建示例如下图：

![image-20201203163632848](/imgs/sandstone_create_pool.png)

### 创建网关

1. 登陆杉岩存储管理界面，从左侧菜单进入”块存储“-->”网关管理“界面。
2. 点击”创建“按钮，选择三个非`ceph mon`角色节点，然后点击”下一步“。
3. 接入点选择“业务平面网络”，点击“确认”进行创建。

# 对接操作

通过ECS界面使用对接包**自动化**对接杉岩存储，无须手动介入。

1. 使用管理员账号登陆ECS控制台。
2. 选择左侧“自动化中心”进入配置界面，选择“高级配置”-->“对接第三方商业存储”。
3. 点击”+“，上传对接包，然后点击加载配置。
4. 等待自动对接完成。

# 对接后验证

## 创建卷类型

需要创建杉岩存储对应的卷类型volume_type后才能使用其存储，具体步骤为：

1. SSH登陆ECS控制节点。
2. 运行`export busy=$(k get po -n openstack|grep busybox|awk '{print $1}'|xargs -L 1);kubectl exec -it ${busy} -n openstack -- bash`进入`busybox`容器。
3. 运行`source openrc`导入环境变量。
4. 运行`openstack volume type create sandstone`创建`sandstone`卷类型。
5. 运行`openstack volume type set --property volume_backend_name=sds-iscsi sandstone`执行卷后端。

## 功能验证

进入ECS控制台界面，验证以下功能：

> 1. 创建云硬盘
> 2. 删除云硬盘
> 3. 挂载云硬盘
> 4. 卸载云硬盘
> 5. 扩展云硬盘
> 6. 创建云硬盘快照
> 7. 删除云硬盘快照
> 8. 通过快照创建云硬盘
> 9. 通过镜像创建云硬盘
> 10. 通过云硬盘创建镜像

# 参考

- [杉岩cinder driver配置指南](https://opendev.org/openstack/cinder/src/branch/master/doc/source/configuration/block-storage/drivers/sandstone-storage-driver.rst)
- [杉岩cinder driver代码](https://opendev.org/openstack/cinder/src/branch/master/cinder/volume/drivers/sandstone)

