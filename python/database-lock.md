---
title: 数据库锁
layout: post
---

## 场景：并发更新

Karbor原生代码更新Plan的Resource是先把Plan的所有Resource置为```deleted```，然后再创建新的创建接口。当并发在Plan在添加同一个资源时，由于没有锁机制，可能导致该资源被添加两次。具体代码如下：
```python
def _plan_resources_update(context, plan_id, resources, session=None):
    session = session or get_session()
    now = timeutils.utcnow()
    with session.begin():
        model_query(
            context,
            models.Resource,
            session=session
        ).filter_by(
            plan_id=plan_id
        ).update({
            'deleted': True,
            'deleted_at': now,
            'updated_at': literal_column('updated_at')
        })
    resources_list = []
    for resource in resources:
        resource['plan_id'] = plan_id
        resource['resource_id'] = resource.pop('id')
        resource['resource_type'] = resource.pop('type')
        resource['resource_name'] = resource.pop('name')
        resource['resource_extra_info'] = resource.pop(
            'extra_info', None)
        resource_ref = _resource_create(context, resource)
        resources_list.append(resource_ref)

    return resources_list
```

解决该问题可以用行锁机制，限制对同一个Plan进行并发更新，具体讲就是Plan更新接口会起一个事务（Python里面是session），这个session会使用```with_for_update```接口查询该Plan，这样在事务结束前，无法并发对该Plan进行查询或修改。具体修改如下：
```python
def plan_update(context, plan_id, values):
    session = get_session()
    with session.begin():
        session.query(models.Plan).filter_by(id=plan_id). \
            with_for_update().first()
        resources = values.get('resources')
        if resources is not None:
            _plan_resources_update(context,
                                   plan_id,
                                   values.pop('resources'),
                                   session=session)

        plan_ref = _plan_get(context, plan_id, session=session)
        plan_ref.update(values)

        return plan_ref
```
