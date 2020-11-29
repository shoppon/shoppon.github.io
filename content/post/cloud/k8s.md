# 快捷命令

删除cinder-volume pods

```python
kubectl get po -n openstack|grep volume|awk '{print $1}'|xargs -I{} kubectl delete po {} -n openstack
```

