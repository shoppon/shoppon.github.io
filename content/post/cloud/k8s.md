---
title: "K8S总结"
categories: ["云计算"]
tags: ["k8s", "容器"]
date: 2020-11-30T20:00:00+08:00
---

# Dashboard

获取登陆token

```shell
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

# 快捷命令

删除cinder-volume pods

```python
kubectl get po -n openstack|grep volume|awk '{print $1}'|xargs -I{} kubectl delete po {} -n openstack
```

