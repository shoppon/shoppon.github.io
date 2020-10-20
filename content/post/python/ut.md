---
title: Python单元测试方法
layout: post
---

## LLT原则
#### 用例一定要用场景和预期
所谓场景，就是用例一定要有目的，比如检查调用cinder网络异常怎么处理，创建快照失败怎么处理。
```python
def test_create_backup_status_error(self, factory_mock):
        """
        场景：创建备份状态为error
        预期：checkpoint和checkpoint item状态均为error，不会出现非稳态现象
        临时快照被清理，临时备份被清理
        副本中fail_reason为cinder backup中的fail_reason
        :return:
        """
```
所谓预期，就是用例一定要有**检查点**，检查点一般是通过断言来表达。**绝对不允许没有检查点的用例**，之前整机备份plugin就有很多这样的反例，
列举一例如下。
```python
def test2(self):
    try:
        backup_service.on_main(checkpoint_object,resource_object,context_object,None)
    except Exception as e:
        self.failIf(plugin_common.PLUGIN_EXCEPTION_STR != e.__str__())
        return
    self.failIf(True)
```
### db可以不用mock，可以使用sqlite memory db，必要时甚至可以使用sqlite本地数据库（方便查看db数据）
```python
def setup_dummy_db():
    options.cfg.set_defaults(options.database_opts, sqlite_synchronous=False)
    CONF.database.connection = 'sqlite:///:memory:'
    CONF.disable_process_locking = True

    engine = get_engine()

    karbor_migration.db_sync()
    migration.db_sync(engine)
    engine = get_engine()
    engine.connect()
```

## Mock原则
#### 如何确定要mock的路径<br>
总结就一句话：**在哪里引用就mock哪个路径**。举例如下：<br>
文件路径：```kangaroo/kangaroo/plugin/volume/backup_operation.py```<br>
```python
from karbor.services.protection.client_factory import ClientFactory
class BackupOperation(protection_plugin.Operation):
    def on_main(self, checkpoint, resource, context, parameters, **kwargs):
        cc = ClientFactory.create_client("cinder", context)
        snap = cc.volume_snapshots.create(volume_id=volume_id)
```

如果想mock ```ClientFactory```就必须```mock kangaroo/plugin/volume/backup_operation.ClientFactory```，而不是```karbor.services.protection.client_factory.ClientFactory```

#### 跨服务调用时mock client，因为LLT运行时周边网络肯定是无法连通的。
```python
@mock.patch("kangaroo.plugin.volume.backup_operation.ClientFactory")
```
#### 长时间任务有轮询操作时**一定要将```interval``` mock掉**，否则LLT执行会很慢。
```python
@mock.patch("kangaroo.plugin.volume.backup_operation.POLL_INTERVAL", 0)
```
#### 异步执行时需要将协程mock，并且捕获协程中的异常，因为协程的异常对主线程不感知。
```python
def mock_spawn(cls, func, *args, **kwargs):
    """
    mock spawn，改成同步调用
    :param func:
    :param args:
    :param kwargs:
    :return:
    """
    try:
        func(*args)
    except Exception:
        # 协程异常对主线程不感知
        pass
```

## Mock技巧
#### 基于类的Mock和基于方法的Mock的区别
```python
@mock.patch("kangaroo.plugin.volume.backup_copy_operation.INTERVAL", 0)
Class TestBackup(BaseTestCase):
    pass
```
```python
@mock.patch("kangaroo.plugin.base_operation.wf_client", mock.MagicMock())
    def __init__(self, *args, **kwargs):
        pass
```

#### mock.MagicMock使用
MagicMock对象可以通过任何属性去访问，返回值是一个子Mock对象，也可以指定该对象的任何属性。
```python
fake = mock.MagicMock()
print fake.a
print fake.b
fake.c = object()
```
#### 使用side_effect构造多次调用不同返回结果
在LLT中，经常遇到需要构造同一个接口多次请求返回不同的场景，这个时候side_effect就能派上用场了。
```python
client_mock.backups.get.side_effect = [Backup(status='creating'),
                                       Backup(status='error'),
                                       Backup(status='error')]
```
在这个例子中，我们模拟了cinder backup调用每次返回不同的状态。
#### 使用```assert_called_with```、```assert_called_with```、```assert_not_called```等断言代码是否被调用。
```python
# 断言快照被清理
client_mock.volume_snapshots.delete.assert_called_once_with(
    snapshot='fake_snapshot_id')
# 断言失败副本被清理
client_mock.backups.delete.assert_called_once_with(
    backup='fake_backup_id1')
```

## LLT中无法mock的坑

#### sqlite功能不全
LLT运行时使用的时sqlite memory db，受限于sqlite功能约束，有些mock无法实现。包括如下：

- 外键约束无法mock：外键不存在时也可以保存。
- 字段长度无限限制：sqlite字段超过列长度定义也可以保存。
- 级联删除无法限制：sqlite当记录有被其他表外键引用时也可以删除。

#### 在LLT代码中不能修改全局的变量
在LLT中不能修改全局变量，否则会影响其他LLT的运行。例举两例，```test_karbor_config_az.py```用例会模拟修改AZ配置文件，修改了全局的AZ配置文件路径，从而导致其他用例因为AZ检查不通过了失败。

配置文件中公有云、私有云的配置，默认是私有云，并且在BaseTestCase的tearDown中会置为私有云。如果需要测试公有云场景，需要手动打开开关。

#### 不能在LLT中修改module
Python的module也是全局的，所有用例都是共用一个Module，因此禁止在LLT在修改Module。反而教材：
```python
ClientFactory.create_client = mock.Mock(
            side_effect=create_client)
        cinder.create = mock.Mock(side_effect=client)
```
这个会导致后面引用ClientFactory的用例失败。
