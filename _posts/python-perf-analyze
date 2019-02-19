# Python性能分析

性能分析的关键是根据系统运行时间统计找出性能瓶颈。Python自带的```CProfile```运行一段代码可生成该代码的运行时间统计的性能文件。

## 性能文件生成

性能文件生成的前提是有一段可独立调用的代码，使用```CProfile```运行该代码：```python -m cProfile -o 256_vm_create_flow.stats perf.py```即可得到性能文件。

Karbor业务代码都是以进程方式运行的，如需针对单个业务操作进行性能分析，要将其改造成独立运行的方式。在这个过程中较复杂的是```Context```构造和运行环境(配置文件、日志、RPC等）准备。

以备份为例，```kangaroo/tests/debug/perf.py```为样例代码。

## 性能文件分析

使用```CProfile```支持交互式地分析性能文件。

#### 名词解释
主要用到的几个名词如下：
```shell
ncalls：表示函数调用的次数；
tottime：表示指定函数的总的运行时间，除掉函数中调用子函数的运行时间；
percall：（第一个percall）等于 tottime/ncalls；
cumtime：表示该函数及其所有子函数的调用运行的时间，即函数开始调用到返回的时间；
percall：（第二个percall）即函数运行一次的平均时间，等于cumtime/ncalls；
filename:lineno(function)：每个函数调用的具体信息；
```

#### 交互式分析

执行```python -m pstats 256_vm_create_flow.stats```进行交互式分析模式。

通过```help```命令可以看到其支持```sort```、```stats```、```strip```、```callees```、```callers```等相关操作。

通过```sort```命令是将性能文件进行指定方式排序，如上表中的名词解释。

通过```stats```命令可输出TOPN条记录。

## 案例

#### 策略绑定虚拟机

优化目标：

1. 64虚拟机并发绑定响应时间响应时间10秒之内。

性能瓶颈：

1. 查询请求较多。

优化思路：

1. 所有网络查询改为并发（默认20），一次性把所有检查依赖的数据查询回来。
2. 去掉RPC调用，在API内完成可保护性检查。
3. 策略插入改成批量执行。
4. 前台预读取下一页。

#### 256虚拟机并发备份

优化目标：

1. 并发执行256虚拟机API响应时间5秒以内。
2. 支持单策略256虚拟机调度。

性能瓶颈：

1. taskflow包含4个task，导致构建taskflow较复杂，时间较长。

优化思路：

1. flow构建改成异步。

2. 任务创建改成批量创建。

#### 100次数据库查询

**注：实测100次查询和1000次查询结果是线性增长的。**

- model_query多个对象的方式

```python
query = model_query(context, cbs_models.CheckPointItem,
                            cbs_models.CheckPoint.provider_id,
                            session=session)
```

- joinedload另外一张表的方式

```python
for col in columns_to_join:
    query = query.options(joinedload(col))
```

1. 在```perf.py```中构造100次查询：

```python
def db_query():
    # 查询100次
    times = 100
    admin_ctx = context.get_admin_context()
    for i in xrange(times):
        db_api.backups_get_all(admin_ctx,
                               filters={'resource_type': 'OS::Nova::Server',
                                        'all_tenants': True},
                               limit=1000, offset=0,
                               sort_keys=None, sort_dirs=None)
```

2. 执行``` python -m cProfile -o 100_backup_query.stats perf.py```生成性能文件。
3. 执行``` python -m pstats 100_backup_query.stats```交互式分析性能文件。
4. 输入```sort cumtime```查询函数进入到返回的时间。
5. 输入```stats 30```查询前30个耗时的函数。

