# 背景

功能列表

是否对接glance、cinder、nova？

控制节点是否安装rpm包

# 杉岩对接

保存cinder配置

```python
kubectl get configmap -n openstack cinder-etc -o yaml > /root/cinder-config-map.yaml
```

保存deployment

```python
kubectl get deployment -n openstack cinder-volume -o yaml > /root/cinder-volume-deployment.yaml

hostpath_volume:
  - name: sandstone-ceph-driver
    host_path: /var/lib/ceph/
      container_path: /var/lib/ceph/
```

修改cinder配置

```python
kubectl edit cm -n openstack cinder-etc

# 在cinder.conf中添加
[DEFAULT]
use_chap_auth = false
enabled_backends = sds-sys, sds-data
backup_ceph_conf = /opt/sandstone/etc/sds/sds.conf
backup_ceph_user = cinder
backup_ceph_chunk_size = 134217728
backup_ceph_pool = backups
backup_ceph_stripe_unit = 0
backup_ceph_stripe_count = 0
restore_discard_excess_bytes = true
# 在backend中添加
[sds-sys]
rbd_flatten_volume_from_snapshot = true
volume_driver = cinder.volume.drivers.rbd.RBDDriver
volume_backend_name = sds-sys
rbd_pool = vms
rbd_ceph_conf = /var/lib/ceph/etc/ceph/pool.conf
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = -1
glance_api_version = 2
[sds-data]
rbd_flatten_volume_from_snapshot = true
volume_driver = cinder.volume.drivers.rbd.RBDDriver
volume_backend_name = sds-data
rbd_pool = volumes
rbd_ceph_conf = /var/lib/ceph/etc/ceph/pool.conf
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
rados_connect_timeout = -1
glance_api_version = 2
```

修改deployment

```python
kubectl edit deployment -n openstack cinder-volume
```

从存储节点拷贝ceph配置文件

```python
scp -r root@192.168.4.61:/var/lib/ceph/etc/ceph/6d199430-f486-40d7-a688-9a1a35e7cf2f.conf /opt/solution/cinder-volume/sandstone
scp -r root@192.168.4.61:/var/lib/ceph/etc/keyring/6d199430-f486-40d7-a688-9a1a35e7cf2f/client/client.admin.keyring /opt/solution/cinder-volume/sandstone
```

添加杉岩的ld.conf，使用`rados`、`rbd`相关so用杉岩的

```python
/var/lib/ceph/etc/ld.so.conf.d/sds-ceph.conf
```

# 问题

rpm包安装后添加软链接失败

```python
ln: failed to create symbolic link ‘/etc/ceph’: File existsdstone/rbdcl
ln: failed to create symbolic link ‘/usr/lib/python2.7/site-packages/ceph_argparse.py’: File exists
ln: failed to create symbolic link ‘/usr/lib/python2.7/site-packages/ceph_argparse.pyc’: File exists
ln: failed to create symbolic link ‘/usr/lib/python2.7/site-packages/ceph_argparse.pyo’: File exists
ln: failed to create symbolic link ‘/usr/lib/python2.7/site-packages/ceph_daemon.py’: File exists
ln: failed to create symbolic link ‘/usr/lib/python2.7/site-packages/ceph_daemon.pyc’: File exists
ln: failed to create symbolic link ‘/usr/lib/python2.7/site-packages/ceph_daemon.pyo’: File exists
ln: failed to create symbolic link ‘/usr/lib/python2.7/site-packages/ceph_volume_client.py’: File exists
ln: failed to create symbolic link ‘/usr/lib/python2.7/site-packages/ceph_volume_client.pyc’: File exists
ln: failed to create symbolic link ‘/usr/lib/python2.7/site-packages/ceph_volume_client.pyo’: File exists
ln: failed to create symbolic link ‘/usr/bin/ceph’: File exists
ln: failed to create symbolic link ‘/usr/bin/rbd’: File exists
ln: failed to create symbolic link ‘/usr/bin/rados’: File exists
```

# 定位

查看连接rados日志

```shell
k logs cinder-volume-7b867c9cb8-46xqr -n openstack|grep -C 10 "_connect_to_rados"
```

