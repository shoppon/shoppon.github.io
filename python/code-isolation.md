---
title: 代码隔离技术
layout: post
---

## 代码隔离技术

#### 目录隔离

目录隔离方法的适用场景必须满足三个条件：

1. 线上线下代码完全不一样。
2. 所隔离的代码没有依赖。
3. 可通过打包过程隔离。

这种方法一般适用于脚本代码，业务代码由于依赖较多，几乎无法适用。目前线上线下的CLI代码是通过该种方法隔离的。

#### 场景开关

线上线下**由于业务场景不一样，导致同一个功能在线上线下不一样**。例如创建策略功能在私有云支持拷贝策略、全备策略，公有云支持复制策略。由于接口、代码是一套，所以只能基于场景做相应判断。当前```cbs_utils.py```已经提供了判断是否是公有云和私有云的公共接口```is_public_scene```和```is_private_scene```，业务代码根据场景做相应适配即可，下面是一个最简单的例子。

```python
if utils.is_private_scene():
    self.check_policy_az_is_supported_protect(service_args)
```

场景开关一种较为高级的实现方式是通过```装饰器```来实现，避免对业务代码过度侵入。整机和卷接口就是通过```cbs_common```中的```set_api_req_type```来区分接口类型。

#### 特性开关

**目前最为常用的隔离手段还是特性开关的方式。**这种方式隔离较为彻底，如果某种场景不需要该特性，直接关闭开关即可，缺点之于代码中可能会存在许多开关逻辑。以流控特性为例：

首先需要在```config.py```中注册相应的特性开关（其实更加建议在业务逻辑中添加开关，所谓的高内聚）：

```python
cfg.BoolOpt('workload_enable',
    default=False,
    help='workload switch'),
```

然后在使用流控的地方根据开关添加相应的业务逻辑：

```python
if workload.is_supported(**kwargs):
    operation_log = kwargs.get('operation_log')
    op_id = operation_log.get('id')
    LOG.debug('Tenant: %s backup resource: %s enter flow, '
              'job_id:%s .' % (context.project_id, resource.id, op_id))
    workload.wait_and_query_job_status(context, op_id, resource)
```

正如前面所讲，这种方式的问题在于可能需要在多处代码中添加开关判断，流控也是如此，需要在所有的正常、异常流程都需要释放流控：

```python
if workload.is_supported(**kwargs):
    workload.update_workload(context, context.project_id)
```



#### 万能对象

特性开关的方式可能存在这样一个问题，需要在很多代码流程中添加判断逻辑，导致代码逻辑非常不连续，到处被割裂。能否支持在在不同特性开关、不同场景下使代码逻辑保持一致呢？答案是可以的，这要求代码中对象的行为在不同场景下是一致的，我们可以通过**万能对象**的方法来实现。

任务特性的隔离就使用这**万能对象**这种隔离方式。任务对象是中```flow```创建过程中创建的，然后在```resource_flow```遍历的时候```inject```到插件中，最终在插件中跟踪任务的状态、进度等。任务特性涉及所有```Karbor```流程代码的修改，总计超过60+文件。如果还是采用传统的特性开关的方式，会导致大量代码中存在```if```、```else```等判断，极其丑陋。那是不是可以在不支持任务的时候创建一个假的任务对象，对外也支持任务对象的所有接口呢？这就是**万能对象**的思维，任务特性正是基于此设计。让我们看一下具体代码实现：

首先我们定义了这样一个对象

```python
class DummyOperationLog(object):
    def __getattr__(self, item):
        if item == 'id':
            return None
        else:
            return self

    def __setattr__(self, key, value):
        return self

    def __getitem__(self, item):
        return self

    def __setitem__(self, key, value):
        return self

    def __call__(self, *args, **kwargs):
        return self
```

我们复写了```__getattr__```、```__setattr__```、```__getitem__```、```__setitem__```和```__call__```方法，这样我们通过```DummyOperationLog()```构造出来的对象可以通过```operation_log.status```、```operation_log['backup']```的方式获取其任意属性，可通过```operation_log.save()```等调用任意方法。

然后在```protect.py```、```restore.py```、```delelte.py```等```flow```创建过程中根据特性开关创建不同的```operation_log```对象

```python
def create_operation_logs(ctx, checkpoint, graph, op_type, restore_id=None):
    op_logs = {}
    for g in graph:
        resource = g.value
        key = resource.type + "#" + resource.id
        if is_support_op_log():
            op_log = utils.create_operation_log(ctx, checkpoint, op_type)
            op_log.restore_id = restore_id
            op_log.extra_info = {
                'resource': {
                    'id': resource.id,
                    'type': resource.type,
                    'name': resource.name
                },
                'common': {
                    'progress': 0,
                    'request_id': ctx.request_id
                }}
            op_log.error_info = {
                'code': '',
                'message': ''
            }
            op_log.status = get_initial_status(ctx, op_type=op_type)
            op_log.save()
        else:
            op_log = DummyOperationLog()
        op_logs.update({key: {"operation_log": op_log.id}})
    return op_logs
```

并且提供```get_op_log```方法获取任务对象，如果没有也返回```DummyOperationLog```：

```python
def get_op_log(context, op_log_id=None):
    if op_log_id is None:
        op_log_object = DummyOperationLog()
    else:
        op_log_object = objects.OperationLog.get_by_id(context,
                                                       op_log_id)
    return op_log_object
```

如此一来，在插件业务层无论哪种场景都可以获取到```operation_log```对象，并且其行为都是一致的，获取任务属性、调用任务方法都不会报错：

```python
self.operation_log.extra_info['backup'] = {
    'checkpoint_item_id': new_item_id,
    'checkpoint_item_name': checkpoint_item.name,
}
self.operation_log.save()
```

综上，这样的代码隔离是不是很炫酷呢？
