---
title: 备份服务团队开发规范
layout: post
---

## 需求管理
所有需要均通过[Vision](https://wa.vision.huawei.com/#/product/domains/7296/requirements/receive/list?conditions=eyJkb21haW5faWQiOjcyOTYsImlucHV0IjpbXSwidXNlciI6Wy0xMF0sImdyb3VwIjp7fX0%3D&view_mode=level&sort_type=updated_time&sort_order=desc&sort_desc=UPDATE_TIME&current_page=1&page_size=50)管理，开发、测试人员只认vision上分解的任务，不接受任何黑活。

## 开源管理
所有对Karbor源码修改须使用patch的方式，不允许直接修改Karbor源码。

Patch生成方法：

- 如果待修改文件之前已经打过Patch，则先执行```git apply kangaroo/src/kangaroo/patches/xxx.patch```应用Patch
- 直接修改Karbor源代码```xxx.py```
- 执行```git diff xxx.py > kangaroo/src/kangaroo/patches/xxx.patch```生成patch，**文件格式必须是目录+文件名**
- 提交patch，回滚Karbor代码修改```git checkout xxx.py```

在出包的时候会调用```git apply kangaroo/src/kangaroo/patches/*```，给Karbor代码打上相应patch。

## 环境规范
- master分支是公共主干，所有代码必须达到TR6水平才能合入
- 发布分支以公私有云+版本号+dev命名，如```public_2.2.0_dev```，发布后打上release标签如```public_2.2.0_release```
- git个人分支必须以姓名缩小开头，便于清理，如```xp_eraser_dev```。
- git个人分支数必须及时清理，允许不超过5个。
- 禁止在git库提交大文件，如需提交必须经过评审。

## 代码合入规范
- 单一MR不得超过500行，如果某个特性超过500行则拆分提交，当前试运作2周

- Commiter每天10:00、17:30、20:00定期检视代码，如无意见则同意MR。

- Commiter提交的检视意见必须当天闭环。

- Commiter每周检视意见低于10条取消Commiter资格(10只是牵引数据，根据实际情况来看)，取消资格的commiter进行例行通报。

- 每周检视意见进入TOP3的非Commiter自动晋升为Commiter。

- 超过三天未合入的MR直接关闭。

- WIP状态的MR不受以上条件约束。

## 用例规范

Karbor所有特性/功能都必须有用例覆盖，所有测试活动都基于用例进行，包括LLT和HLT。

#### 用例ID

用例ID规格为**TC+CBR+对象+操作+类型+场景+编号**，例如TC_CBR_Backup_Query_Server_001。

用例对象列表如下表：

| 对象     | 名称        |
| -------- | ----------- |
| 任务     | Op          |
| 可保护性 | Protectable |
| 存储库   | Vault       |
| 还原点   | Checkpoint  |
| 备份     | Backup      |
| 策略     | Policy      |
| 项目     | Project     |
| 计量     | Meter       |
| 运营     | Billing     |

#### 用例划分


#### 用例管理

- 用例基于[云龙CloudTest](https://clouddragon.huawei.com/cloudtest/cloudtest/newFeature?serviceId=ccaeb9cbf825419dbacc897898e8a925!versionUri~267qlf5q4kt!versionName~20190130_CBR_SIT5)管理，用例通过tags字段标记是否需要实现LLT和是否已经实现LLT。

- TSE和SE根据用例的场景及重要性标记需要实现LLT的用例，在tags字段中增加**LLT_NEED**标签。特性开发按照标签搜索出需要实现**LLT**的用例，实现LLT后在tags中添加**LLT_COMPLETE**标签。

- 开发的实现LLT时需要在用例上添加```@base.testcase```的装饰器，例如```@base.testcase('TC_CBR_Backup_Query_Server_001', author='x00250203')```，标记用例ID和用例负责人。

#### 开发测试协同

- 开发根据自己的代码编写LLT时发现没有HLT，而且是重要场景时，需要在云龙上添加用例，或者修改原有用例补全场景。如果需要测试补充测试步骤和用例，则创建或复制一个空用例，在tags中添加**TC_NEED**标签。

- 测试搜索tags中包含**TC_NEED**标签，完善用例。

#### 流程规范

- 在特性**转测前一周**，TSE和结对测试需要完成测试用例输出和评审，评估重要场景并且标记出需要实现LLT的用例。

- 转测试后测试提单必须提交用例ID，如果无用例必须先补充用例。

- 迭代结束前用例库中不能存在上个迭代的**TC_NEED**用例。

#### 职责划分

- TSE对整体用例质量负责，保证用例无重要场景遗漏，合理划分用例维度，精简用例结构。
- SE/TL对整体代码质量负责，代码检视时保证重要场景均有LLT覆盖。

## 编码规范 
- [PEP8](http://www.jianshu.com/p/52f4416c267d)问题必须清零

- 所有python、shell文件格式UTF-8，换行符LF，不能使用CRLF。

- shell脚本必须使用严格模式，不允许出错

  ```shell
  set -o errexit # 只要出错就退出
  set -o nounset # 不允许引用不存在的变量
  set -o pipefail # 各管道内不允许出错
  ```

## 错误码
- 错误码按照[错误码规范](http://office.huawei.com/sites/it-rdsdmd/scspdu/drdd/_layouts/15/WopiFrame.aspx?sourcedoc={57C4CBC4-BD0C-4C42-8A0F-222C76B2E42C}&file=%E9%94%99%E8%AF%AF%E7%A0%81%E5%88%97%E8%A1%A8.xlsx&action=default)申请合适的错误码
- 每个错误码必须对应代码中一个异常。
- 每个异常业务场景对应一个Exception，一个错误码，不允许定义笼统的异常（参数校验除外）。备份失败、删除失败等笼统异常是不允许的。
- 错误码异常必须按照错误码序号进行排列。
- 异常的错误信息在异常类通过message或者v3_message定义，抛异常时仅传参数，禁止整个异常信息由外部输入。

## CI规范
- 代码合入前必须跑Flake8+LLT门禁。
- LLT时间不允许超过3分钟。

## 参数检验
- 参数合法性必须使用jsonschema框架进行校验，不需要定义错误码。
- 参数逻辑检验在代码中进行

## 业务编码
- 原则上接口必须定义ViewBuilder，屏蔽模型修改对接口的影响。
- 对数据库操作必须使用objects进行封装，代码调用不感知数据库层。

## LLT
- 必须编写LLT用例，用例有明确的**场景**和**检查点**。
- 单元测试包目录结构要和源代码的包目录保持一致。
- 原则上代码LLT覆盖率需达到90%以上。
- 不允许修改全局变量、Module、配置。
- 异步调用必须Mock。
- Sleep必须Mock。

## 日志
- 日志路径必须[规范](日志规范)
- 异常必须打印堆栈
- 调用外部(nova、cinder、workflow、obs、告警)非查询接口必须打印日志
- 业务请求必须打印req_id，异步调用时需要wrap_context

## 接口
- 接口必须而且仅支持APIManager进行管理，不允许直接Word等非结构化方式管理。
- 接口参数必须有模型对应。
- 模型命令按照Model+Op+Req/Resp的格式进行定义。
- 接口定义参考[IaaS Yaml定义规范](http://3ms.huawei.com/km/groups/3467417/blogs/details/5796163?moduleId=)。
- 接口和模型的变更必须进行经过评审。
- 告警、错误码变更必须进过评审

## 安全
- 所有API请求的入口，包括查询接口都需要进行参数校验。参数的个数、类型、取值范围需要和API文档保持一致。
- 所有API请求的入口，包括查询接口都需要对请求进行token认证和RBAC权限校验。
- 编码考虑Python主流版本的兼容性
  - 使用```six.text_type```，禁用```unicode```。
  - 使用```six.string_type```，禁用```basestring```。
  - 使用```next(iterator)```，禁用```iterator.next()```。
  - 使用```six.iteritems(dict)```，禁用```dict.iteritems()```。
  - 使用```data.decode('utf-8')```，禁用```data.decode()```。
  - 使用```except x as y```，禁用```except x，y```
- 进行加、减、乘、除运算时，必须明确变量的数据类型
- 变量赋值前，先确认变量的数据是同一种类型
- 异常场景必须考虑资源的释放
- 禁止滥用锁资源，并发场景需要防止死锁

## 不要重复发明轮子

- 解析json统一使用```oslo_serialization```
- 日志打印统一使用```oslo_log```
- 配置管理统一使用```oslo_config```
- uuid统一使用```oslo_utils```中的uuidutils
- db操作操作```sqlalchemy```和```oslo_db```
- 权限控制使用```oslo_policy```

## 编码风格

### 代码编排

- 缩进。采用4个空格的缩进，不使用Tap，更不能混合使用Tap和空格。
- 每行最大长度79，换行可以使用反斜杠，最好使用圆括号。换行点要在操作符的后边敲回车。
- 类和top-level函数定义之间空两行；类中的方法定义之间空一行；函数内逻辑无关段落之间空一行；其他地方尽量不要再空行。

- 模块```import```顺序。模块内容的顺序：按标准、三方和自研顺序依次排放，之间空一行。
- 禁止在一句import中多个库，比如在同一语句中import os, sys，而是分别执行import。

### 空格的使用

- 各种右括号前不要加空格。
- 逗号、冒号、分号前不要加空格。
- 函数的左括号前不要加空格。如func(1)。
- 序列的左括号前不要加空格。如list[2]。
- 操作符左右各加一个空格，不要为了对齐增加空格。
- 函数默认参数使用的赋值符左右省略空格。
- 不要将多句语句写在同一行。
- if/for/while语句中，即使执行语句只有一句，也必须另起一行。

### 命名规则

- 所有的类采用驼峰式命名，每个单词的首字母大写，类名的命名体现类的作用。
- 局部变量的命名全部小写，每个单词之间使用_分割，变量的命名准确体现变量的含义。
- 全局变量的命名全部大写，每个单词之间使用_分割。
- 数据库方法的命名，采用操作对象_方法的方式，例如创建plan，操作对象是plan，方法是create。

备份服务统一编码规范[模板文件路径](http://code.huawei.com/csbs/dev/blob/master/tools/pycharm/csbs_standard_codestyle.jar)，使用方法：

1. 打开pycharm
2. 选择```File```-->```Import Settings```，指定统一的模板文件
3. 提交代码前必须格式化，选择```Code```-->```Reformt Code```，建议使用快捷键```Ctrl+Alt+L

## 做一个有技术情结的软件工程师

- **Write down background**。特性开发编写技术文档，记录周边依赖（接口、架构、流程），记录方案背景，记录关键实现。
- **Don't Repeat Yourself**。杜绝Copy/Paste，当修改一个功能需要重复修改多个地方请重构它！
- **Clean your code**。严格遵从代码规范，做到代码无任何警告。
- **Do your best**。实现一个功能，修改一个问题的方式有很多种，尽你最大努力找到最优雅的解法，让代码出现在最合适的地方。架构的腐烂源于随意的修改。
- **Refactor anytime, anywhere**。日志打印不清晰？告警无法定位？随时随地重构你的代码，把优化做在平常。
