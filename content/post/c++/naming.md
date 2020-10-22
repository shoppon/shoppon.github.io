---
title: "命名"
categories: ["c++"]
tags: [""]
date: 2020-07-17T19:26:38+08:00
---

# 命名

## 命名方法

### 返回真伪值的方法

| 位置   | 单词   | 意义                                                         | 例            |
| ------ | ------ | ------------------------------------------------------------ | ------------- |
| Prefix | is     | 对象是否符合期待的状态                                       | isValid       |
| Prefix | can    | 对象**能否执行**所期待的动作                                 | canRemove     |
| Prefix | should | 调用方执行某个命令或方法是**好还是不好**,**应不应该**，或者说**推荐还是不推荐** | shouldMigrate |
| Prefix | has    | 对象**是否持有**所期待的数据和属性                           | hasObservers  |
| Prefix | needs  | 调用方**是否需要**执行某个命令或方法                         | needsMigrate  |

###  用来检查的方法

| 单词     | 意义                                                 | 例             |
| -------- | ---------------------------------------------------- | -------------- |
| ensure   | 检查是否为期待的状态，不是则抛出异常或返回error code | ensureCapacity |
| validate | 检查是否为正确的状态，不是则抛出异常或返回error code | validateInputs |

### 按需求才执行的方法

| 位置   | 单词      | 意义                                      | 例                     |
| ------ | --------- | ----------------------------------------- | ---------------------- |
| Suffix | IfNeeded  | 需要的时候执行，不需要的时候什么都不做    | drawIfNeeded           |
| Prefix | might     | 同上                                      | mightCreate            |
| Prefix | try       | 尝试执行，失败时抛出异常或是返回errorcode | tryCreate              |
| Suffix | OrDefault | 尝试执行，失败时返回默认值                | getOrDefault           |
| Suffix | OrElse    | 尝试执行、失败时返回实际参数中指定的值    | getOrElse              |
| Prefix | force     | 强制尝试执行。error抛出异常或是返回值     | forceCreate, forceStop |

### 异步相关方法

| 位置            | 单词         | 意义                                         | 例                    |
| --------------- | ------------ | -------------------------------------------- | --------------------- |
| Prefix          | blocking     | 线程阻塞方法                                 | blockingGetUser       |
| Suffix          | InBackground | 执行在后台的线程                             | doInBackground        |
| Suffix          | Async        | 异步方法                                     | sendAsync             |
| Suffix          | Sync         | 对应已有异步方法的同步方法                   | sendSync              |
| Prefix or Alone | schedule     | Job和Task放入队列                            | schedule, scheduleJob |
| Prefix or Alone | post         | 同上                                         | postJob               |
| Prefix or Alone | execute      | 执行异步方法（注：我一般拿这个做同步方法名） | execute, executeTask  |
| Prefix or Alone | start        | 同上                                         | start, startJob       |
| Prefix or Alone | cancel       | 停止异步方法                                 | cancel, cancelJob     |
| Prefix or Alone | stop         | 同上                                         | stop, stopJob         |

###  回调方法

| 位置   | 单词   | 意义                       | 例           |
| ------ | ------ | -------------------------- | ------------ |
| Prefix | on     | 事件发生时执行             | onCompleted  |
| Prefix | before | 事件发生前执行             | beforeUpdate |
| Prefix | pre    | 同上                       | preUpdate    |
| Prefix | will   | 同上                       | willUpdate   |
| Prefix | after  | 事件发生后执行             | afterUpdate  |
| Prefix | post   | 同上                       | postUpdate   |
| Prefix | did    | 同上                       | didUpdate    |
| Prefix | should | 确认事件是否可以发生时执行 | shouldUpdate |

### 操作对象生命周期的方法

| 单词       | 意义                           | 例              |
| ---------- | ------------------------------ | --------------- |
| initialize | 初始化。也可作为延迟初始化使用 | initialize      |
| pause      | 暂停                           | onPause ，pause |
| stop       | 停止                           | onStop，stop    |
| abandon    | 销毁的替代                     | abandon         |
| destroy    | 同上                           | destroy         |
| dispose    | 同上                           | dispose         |

### 与集合操作相关的方法

| 单词     | 意义                         | 例         |
| -------- | ---------------------------- | ---------- |
| contains | 是否持有与指定对象相同的对象 | contains   |
| add      | 添加                         | addJob     |
| append   | 添加                         | appendJob  |
| insert   | 插入到下标n                  | insertJob  |
| put      | 添加与key对应的元素          | putJob     |
| remove   | 移除元素                     | removeJob  |
| enqueue  | 添加到队列的最末位           | enqueueJob |
| dequeue  | 从队列中头部取出并移除       | dequeueJob |
| push     | 添加到栈头                   | pushJob    |
| pop      | 从栈头取出并移除             | popJob     |
| peek     | 从栈头取出但不移除           | peekJob    |
| find     | 寻找符合条件的某物           | findById   |

### 与数据相关的方法

| 单词   | 意义                                   | 例            |
| ------ | -------------------------------------- | ------------- |
| create | 新创建                                 | createAccount |
| new    | 新创建                                 | newAccount    |
| from   | 从既有的某物新建，或是从其他的数据新建 | fromConfig    |
| to     | 转换                                   | toString      |
| update | 更新既有某物                           | updateAccount |
| load   | 读取                                   | loadAccount   |
| fetch  | 远程读取                               | fetchAccount  |
| delete | 删除                                   | deleteAccount |
| remove | 删除                                   | removeAccount |
| save   | 保存                                   | saveAccount   |
| store  | 保存                                   | storeAccount  |
| commit | 保存                                   | commitChange  |
| apply  | 保存或应用                             | applyChange   |
| clear  | 清除数据或是恢复到初始状态             | clearAll      |
| reset  | 清除数据或是恢复到初始状态             | resetAll      |

### 成对出现的动词

| 单词           | 意义              |
| -------------- | ----------------- |
| get获取        | set 设置          |
| add 增加       | remove 删除       |
| create 创建    | destory 移除      |
| start 启动     | stop 停止         |
| open 打开      | close 关闭        |
| read 读取      | write 写入        |
| load 载入      | save 保存         |
| create 创建    | destroy 销毁      |
| begin 开始     | end 结束          |
| backup 备份    | restore 恢复      |
| import 导入    | export 导出       |
| split 分割     | merge 合并        |
| inject 注入    | extract 提取      |
| attach 附着    | detach 脱离       |
| bind 绑定      | separate 分离     |
| view 查看      | browse 浏览       |
| edit 编辑      | modify 修改       |
| select 选取    | mark 标记         |
| copy 复制      | paste 粘贴        |
| undo 撤销      | redo 重做         |
| insert 插入    | delete 移除       |
| add 加入       | append 添加       |
| clean 清理     | clear 清除        |
| index 索引     | sort 排序         |
| find 查找      | search 搜索       |
| increase 增加  | decrease 减少     |
| play 播放      | pause 暂停        |
| launch 启动    | run 运行          |
| compile 编译   | execute 执行      |
| debug 调试     | trace 跟踪        |
| observe 观察   | listen 监听       |
| build 构建     | publish 发布      |
| input 输入     | output 输出       |
| encode 编码    | decode 解码       |
| encrypt 加密   | decrypt 解密      |
| compress 压缩  | decompress 解压缩 |
| pack 打包      | unpack 解包       |
| parse 解析     | emit 生成         |
| connect 连接   | disconnect 断开   |
| send 发送      | receive 接收      |
| download 下载  | upload 上传       |
| refresh 刷新   | synchronize 同步  |
| update 更新    | revert 复原       |
| lock 锁定      | unlock 解锁       |
| check out 签出 | check in 签入     |
| submit 提交    | commit 交付       |
| push 推        | pull 拉           |
| expand 展开    | collapse 折叠     |
| begin 起始     | end 结束          |
| start 开始     | finish 完成       |
| enter 进入     | exit 退出         |
| abort 放弃     | quit 离开         |
| obsolete 废弃  | depreciate 废旧   |
| collect 收集   | aggregate 聚集    |

### 专有名词

| 场景 | 名词     |
| ---- | -------- |
| 引擎 | Engine   |
| 分发 | Dispatch |
| 引用 | Invoke   |

