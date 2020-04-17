## 背景

云上云下的卷备份、虚拟机备份总共有3套构架：配套FusionCloud 2.0的DPS卷备份、配套FusionCloud 2.05的Karbor整机备份、配套FusionCloud 6.0的VBS卷备份和Karbor整机备份，整体维护成本很高。

## 关键设计

#### 割接流程

总体上分为4步：

1. 全量割接
2. 流量倒换
3. 增量割接
4. 任务续做

#### 割接数据有效性，副本可恢复、复制、删除，策略可执行、自动调度、修改、删除。

根据VBS/DPS数据结构，定义python orm模型，将VBS/DPS数据通过sqlalchemy读取到内存，然后根据定义好的规则转换成Karbor数据。具体转换规则见下面章节。

#### 割接过程无临时数据、垃圾数据、残留数据产生。

整个数据割接使用一个事务，成功则提交事务，失败则回滚事务。

VBS/DPS数据导入到```cinder```、```autobackup```、```dps```数据库，不直接导入到karbor数据库，不产生中间数据，垃圾数据。

#### 数据割接使用DMK，参数从界面输入，一键完成割接。

整个割接过程包括以下几个步骤：

- 上传工具包
- 使用ssh工具自动登陆到目标节点
- 从cinder/autobackup/dps导出目标数据库
- 将目标数据库自动下载到karbor节点
- 将目标数据库导入到karbor节点数据库
- 将目标数据转换成karbor数据结构
- 重启operationengine进程

所有步骤除输入参数外，其余一键完成。

#### 支持一键回滚割接数据，支持多次割接数据。

所有从vbs/dps导入过来的数据都会打上标签。

vbs副本使用```0.5版本号，策略描述为```imported_from_vbs```。

dps副本使用```0.4版本号，策略描述为```imported_from_dps```。

回滚时通过标记一键删除，不残留数据。

#### 支持脚本方式清除所有快照和副本

如果客户不需要保留快照或者副本，可通过脚本一键清除快照和副本。

脚本支持流控，一次最多下发10个任务，最多100个任务同时执行。

脚本支持重试10次，最大超时时间2天。

#### 自动dump数据库

提供```db_mover.py```脚本，交互式输入密码，自动登陆```cinder```、```autobackup```、```apicom```节点，执行```gs_dump```导出指定数据库表，然后通过```sftp```下载到Karbor节点，然后导入到数据库。

## 约束和限制

- 必须将Karbor升级到统一构架版本，并且服务正常运行。Karbor统一构架版本仅适配FusionCloud 6.3版本。
- Karbor节点必须与待收编的VBS数据库节点、DPS节点、Cinder数据库节点SSH连通，并且获取目标节点登陆用户名、密码、超级用户密码、数据库```gaussdba```用户密码。
- 割接之前需将VBS/DPS进程停止，**不支持带业务割接**，需保证无业务正在运行，**非稳态数据不支持割接**。
- 解决方案基础组件如openstack、manage one、iam等必须先升级并且完成割接。（**主要是用户数据**）
- DPS割接之后不再提供计量接口，话单继承Karbor话单。

## 版本配套表

| 组件        | DC2 2.0                               | FusionCloud/NFVI 2.06                                        | FusionCLoud 6.0/6.1                                          |
| ----------- | ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| DPS         | OceanStor DJ V1R1C20                  | NA                                                           | NA                                                           |
| VBS自动备份 | NA                                    | NA                                                           | Autobackup 2.2.0/Autobackup 2.2.1                            |
| Karbor      | NA                                    | OceanStor BCManager V200R001C10SPC200<br>OceanStor BCManager V200R001C10SPC100<br>OceanStor BCManager V200R001C10SPC202<br>OceanStor BCManager V200R001C10SPC205 | OceanStor BCManager V200R001C10SPC200<br>OceanStor BCManager V200R001C10SPC100<br>OceanStor BCManager V200R001C10SPC202<br>OceanStor BCManager V200R001C10SPC205 |
| eBackup     | OceanStor BCManager V200R001C10SPC200 | OceanStor BCManager V200R001C10SPC200<br>OceanStor BCManager V200R001C10SPC100<br>OceanStor BCManager V200R001C10SPC202<br>OceanStor BCManager V200R001C10SPC205 | OceanStor BCManager V200R001C10SPC200<br>OceanStor BCManager V200R001C10SPC100<br>OceanStor BCManager V200R001C10SPC202<br>OceanStor BCManager V200R001C10SPC205 |

## VBS收编

#### 模型

| 模型     | Autobackup/Cinder  | Karbor             |
| -------- | ------------------ | ------------------ |
| 计划     | BackupPolicy       | Plan               |
| 资源     | PolicyResource     | Resource           |
| 策略     | BackupPolicyExtend | ScheduledOperation |
| 调度时间 | BackupTrigger      | Trigger            |
| 副本     | Backup             | CheckPointItem     |

#### 备份副本从cinder割接

cinder backup记录了备份副本的**名称**、**状态**、**描述**、**副本大小**、**快照id**、**源卷id**、**源卷大小**、**创建时间**、**更新时间**等属性，backup的service_metadata记录了卷副本**是否增备**、**平均速率**、**空间节省率**、**副本类型（备份、复制）**、**是否启动盘**等属性。

#### 备份策略相关从autobackup割接

#### 备份策略格式转换

VBS调度策略使用的格式为```quartz```库的[```cron```格式](https://www.jianshu.com/p/f03b1497122a)。目前支持两种方式，一种按天，一种按周。

Karbor的调度策略使用的是```icalendar```库，支持[```RFC2445```格式](https://tools.ietf.org/html/rfc2445)。

**如果未设置时间默认置为00:00，若未设置间隔默认置为14天**

Karbor的调度策略每隔N天不支持从每月1号开始，比如每隔两天，1月31号调度之后2月1号不会调度，2月2号才会调度，但是VBS的调度是从每月1号开始，即2月1号又会调度。因此，从VBS迁移过来的开始统一设置为**2018-1-1 00:00:00**。

#### 约束和限制

- 任务相关不割接，副本的copy_status为na，**有可能会导致多复制一次**。
- VBS未保存卷名，所以resource和checkpoint_item的resource_name使用resource_id。
- VBS任务信息（保存在组合API中）不割接，收编之后无法查看历史备份、复制任务。
- VBS配额数据不割接，收编之后使用Karbor配额接口。Karbor会统计VBS使用量，重新生成配额信息。

#### 格式

时间格式由两个字段保存，**时间点**和**天数**，调度时间由这两个值相乘。

Console在UTC+0800时区，选择星期一的07:00和09:00，下发时减去时区偏移量，但天数不变，所以下发的值为**周一的23:00和01:00**。

VBS数据库中存的时间实际上是不准的，需要加上服务器时间进行转换。比如之前策略的**时间点**是1:00，23:00，天数是周一，服务器时区是**UTC+0800**。实际上并不能真正在周一的1:00和23:00调度，而是要**加上时区偏移量再减去24**，即需要在周一的01:00（01:00+8< 24，说明在当天）和周日的23:00（23:00+8>24，大于24说明要往前推1天）调度。

如果Console的时区和服务器时区不一致，会导致服务端还原时间点出错。针对该问题，接口添加时区参数，Console创建时将用户时区下发。

割接时需要对策略调度时间做如下处理：

1. 获取autobackup服务器时区。
2. 获取用户创建策略的时区。
3. 获取还原策略时间的时区，如果用户设置了则以用户设置的为准，否则取autobackup服务器时区。
4. 还原调度时间点，将数据库存储的时间点加上前面步骤获取的时区，并视时区进行**加减24**操作，得到用户真正的时间点（UTC时间）。
5. 将第4点获取的用户时间点还原成UTC时间点，此时有可能会产生跨天的情况。对于前面的例子，即转换成周日的23:00和周一的01:00。

#### 接口兼容

割接时将时区信息保存在pattern里面，接口兼容时根据时区反解成原VBS格式。

## DPS收编

#### 模型

DPS的模型与Karbor模型较为类似，对应关系如下：

| 模型     | DPS                                | Karbor             |
| -------- | ---------------------------------- | ------------------ |
| 计划     | ProtectionPlan                     | Plan               |
| 资源     | ProtectionObject                   | Resource           |
| 策略     | ProtectionPolicy, PolicyExtraSpecs | ScheduledOperation |
| 调度时间 | ScheduleTimer, AsyncCopyPolicy     | Trigger            |
| 副本     | BackupTransactionItem              | CheckPointItem     |

#### 策略割接规则

DPS的策略分为三种：**本地**、**远端**、**本地和远端**。

本地是指仅备份到本地。

远端是指直接备份到远端。

本地和远端是指先备份到本地，然后再复制到远端。

策略割接时，**本地和远端策略均只有备份调度周期**，本地和远端策略有备份调度周期和复制调度周期。由于DPS的复制调度时间是全局的复制周期，所以割接时每个策略都添加该全局复制周期。

#### 约束和限制

- 一致性组不支持
- DPS未保存卷名，所以resource和checkpoint_item的resource_name使用resource_id
- 无调度时间的备份策略（手工备份策略）不割接
- DPS策略过期时间之前由ManageOne实现，过期时间不割接。

## 常用命令

#### 导出cinder backup数据库

```/opt/gaussdb/app/bin/gs_dump cinder -t backups -t quotas -t quota_usages -W FusionSphere123 -f /home/gaussdba/cinder.sql```

#### 导出vbs(组合API数据)

```/opt/gaussdb/dbprogram/bin/gs_dump taskmgr -t taskmgr_job -W Manager@123 -f /home/dbadmin/taskmgr.sql```

#### 导出autobackup数据库

```/opt/gaussdb/dbprogram/bin/gs_dump autobackup -t backup_policy -t backup_policy_extend -t backup_trigger -t backup_relation_policy -t policy_resource -t tag -t backup_create_running_job_record -t backup_create_waiting_job_record -W Manager@123 -f /home/dbadmin/vbs.sql```

#### 导出dps数据

```/opt/gaussdb/app/bin/gs_dump dps -t protection_plan -t protection_object -t protection_policy -t protection_plan_policy_mapping -t schedule_timer -t backup_transaction_item -t policy_extra_specs -t backup_target_object -t async_copy_policy  -W CloudService@123! -f /home/gaussdba/dps.sql```

#### 任务割接

VBS回调Karbor是以taskmgr_job的job_id作为参数。

第一次增量（30分钟）-->倒量（10分钟）-->第二次增量（30分钟）-->同步状态。

逻辑：**以VBS的job状态为准**。

1. 第一次增量割接过程中产生的数据在Karbor中不存在，流量倒换后VBS回调到Karbor，会导致更新任务失败。解决办法：新增**同步状态步骤**，查询autobackup中的running_job，关联查询其vbs的job，如果其状态变为稳态则同时改成稳态。
2. 在流量倒换后VBS回调到Karbor，Karbor改成稳态，第二次增量割接autobackup又将状态同步成了非稳态。解决办法：增量割接前删除autobackup和vbs中在Karbor改过状态的job，不增量割接。

#### 续做非稳态数据

```docker exec karborprotection bash -c "kangaroo-manage continuous_vbs"```

## 割接指导

- 若环境中存在karbor（DC2 2.05和DC2 6.0），则将karbor升级到统一架构版本，若无karbor（DC2 2.0），则全新安装。具体安装和升级指导见DFX裁剪版本指导。
- 登陆DMK，导入数据割接工具包，选择操作选项。
- 配置节点参数，参数顾名思义。```karbor```节点为数据库主节点，割接VBS时不需要填```dps```信息；割接DPS时```autobackup```和```fsp```信息
- 配置Karbor节点参数。
- 一键执行，**enjoy it!**

### 验证策略

通过```insert-select```方法注入5W条数据```INSERT INTO BACKUPS (ID, CREATED_AT, UPDATED_AT, DELETED_AT, DELETED, VOLUME_ID, USER_ID, PROJECT_ID, HOST, AVAILABILITY_ZONE, DISPLAY_NAME, DISPLAY_DESCRIPTION, CONTAINER, STATUS, FAIL_REASON, SERVICE_METADATA, SERVICE, SIZE, OBJECT_COUNT, PARENT_ID, TEMP_VOLUME_ID, TEMP_SNAPSHOT_ID, NUM_DEPENDENT_BACKUPS, SNAPSHOT_ID, DATA_TIMESTAMP, RESTORE_VOLUME_ID) select generate_series(1,50000),'2017-11-08 11:32:02.35061','2017-11-08 11:33:19.93635',null,'f','2647a649-3b5b-4dfa-be72-f86a9ee45d75','9e3ec9f874fa43f0abca5ff2eb207718','25a16f26dbc84b3c8b4984caafe4ad98','az1.dc1.002','az1.dc1','mock','{"ST":0,"INC":1}',null,'available','','{"DL": 2, "VK": "", "bootable": true, "SP": "04b733ae-ffa4-49eb-8bdf-86efe27af825/0a02ccf1-00c9-4481-8159-c6ae2c8e768f/2647a649-3b5b-4dfa-be72-f86a9ee45d75", "backupurl": "97458756-97a4-4e8a-8057-8908bb58e42a", "ST": 0, "BT": "2", "SS": 90, "BP": "192.168.41.2:/csbsPrivateBackup", "CMKID": "", "progress": "0", "CS": 642, "VT": "pool1", "Type": 1, "ebk_T_I": "5ee362bd-2bc7-4bca-98bc-760dd0774da2", "AT": 1.2}','cinder.proxy.backup.drivers.cascaded',50,642,null,null,null,null,'0d663291-49fa-4231-8df3-0bf5613cdb52','2017-11-08 11:31:29.052949',null;```

通过正则表达删除测试数据delete from BACKUP_CREATE_RUNNING_JOB_RECORD where job_id ~ '^\d{6,6}$';

53400条数据实测时间消耗**4分13秒**，内存占用**560M**，CPU占用**单核75%**。

数据构造

```sql
INSERT INTO BACKUPS (ID, CREATED_AT, UPDATED_AT, DELETED_AT, DELETED, VOLUME_ID, USER_ID, PROJECT_ID, HOST, AVAILABILITY_ZONE, DISPLAY_NAME, DISPLAY_DESCRIPTION, CONTAINER, STATUS, FAIL_REASON, SERVICE_METADATA, SERVICE, SIZE, OBJECT_COUNT, PARENT_ID, TEMP_VOLUME_ID, TEMP_SNAPSHOT_ID, NUM_DEPENDENT_BACKUPS, SNAPSHOT_ID, DATA_TIMESTAMP, RESTORE_VOLUME_ID) select generate_series(1,100000),'2017-11-08 11:32:02.35061','2017-11-08 11:33:19.93635',null,'f','2647a649-3b5b-4dfa-be72-f86a9ee45d75','9e3ec9f874fa43f0abca5ff2eb207718','25a16f26dbc84b3c8b4984caafe4ad98','az1.dc1.002','az1.dc1','mock','{"ST":0,"INC":1}',null,'available','','{"DL": 2, "VK": "", "bootable": true, "SP": "04b733ae-ffa4-49eb-8bdf-86efe27af825/0a02ccf1-00c9-4481-8159-c6ae2c8e768f/2647a649-3b5b-4dfa-be72-f86a9ee45d75", "backupurl": "97458756-97a4-4e8a-8057-8908bb58e42a", "ST": 0, "BT": "2", "SS": 90, "BP": "192.168.41.2:/csbsPrivateBackup", "CMKID": "", "progress": "0", "CS": 642, "VT": "pool1", "Type": 1, "ebk_T_I": "5ee362bd-2bc7-4bca-98bc-760dd0774da2", "AT": 1.2}','cinder.proxy.backup.drivers.cascaded',50,642,null,null,null,null,'0d663291-49fa-4231-8df3-0bf5613cdb52','2017-11-08 11:31:29.052949',null;
```

```sql
INSERT INTO BACKUP_POLICY (POLICY_ID, CREATED_AT,DOMAIN_ID,EXTRA_INFO,BACK_UP_FREQUENCY,POLICY_NAME,POLICY_STATUS,REMAIN_FIRST_BACKUP_OF_CUR_MONTH,RENTENTION_DAY,RENTENTION_NUM,SCHEDULE_TASK_ID,BACK_UP_START_TIME,TENANT_ID,UPDATED_AT,WEEK_FREQUENCY ) select generate_series(1,100000),'2018-08-10 18:21:51.528','ba5ddc5a66cf484e8a70f450ffebc3ed',null,1,'test',0,'t',null,14,'autobackup_task_info.f19935ca-486b-4cd9-a8e4-45c392f65db7','15:00','601240b9c5c94059b63d484c92cfe309','2018-08-11 18:21:51.528',null;
```

```sql
INSERT INTO TASKMGR_JOB (JOB_ID,BEGIN_TIME,CONTEXT,CREATED_AT,CREATED_BY,CURRENT_RUN_TYPE,CURRENT_STATUS,DELAY_TIME,END_TIME,EXECUTION_STATUS,FAIL_REASON,JOB_DEF_NAME,JOB_TYPE,LOOP_NUM,ORDER_ID,REQ_CONTEXT,PRIO,REQUEST,RESPONSE,RESPONSE_CODE,RESPONSE_TIME,SCHEDULED_AT,SERVER_HOSTNAME,SKIP_TASKS,SUB_RUN_TYPE,TASK_INDEX,TASK_NUM,TENANT_ID,TOKEN_ID,UPDATED_AT) select generate_series(300000,305000),'2018-10-18 19:36:05.082',null,'2018-10-18 00:00:00',null,'EXECUTE','SUCCESS',0,'2018-10-18 19:39:16.668','SUCCESS',null,'bksDeleteBackup','LONG',0,null,null,1,null,null,null,null,'1539862756668','AZ1-VBS-SRV02',null,'DO',1,1,'d678a3568ff04af6a2487336c4e0d1ed','40c89b186641d2cb016688ad84d904da','1539862756676';
```

```sql
INSERT INTO  BACKUP_CREATE_WAITING_JOB_RECORD (JOB_ID,AVAILABILITY_ZONE,CREATED_AT,CURRENT_RUN_STATE,DOMAIN_ID,NEXT_FIRE_TIME,POLICY_ID,PRIORITY,RESOURCE_ID,RESOURCE_TYPE,TENANT_ID,UPDATED_AT,METADATA,OWNED_BY) select generate_series(200001,202000),'pod0.eu-west-0b','2018-10-23 20:00:00.505','EXECUTE_SUCCESS','0316914de49845e281f80eb8b3be94db','2018-07-19 15:25:10.973','4daad27d-864b-47f0-bc0d-8f4e2abcc2ea',0,'457e9c2a-d161-4e5b-b803-51e8c041f395','volume','40c89b186641d2cb0166a0ccf306068e','2018-10-23 20:03:04.001',null,'1';
```

```sql
INSERT INTO  BACKUP_CREATE_RUNNING_JOB_RECORD (JOB_ID,AVAILABILITY_ZONE,BACKUP_NAME,CREATED_AT,CURRENT_RUN_STATE,DOMAIN_ID,FAIL_REASON,POLICY_ID,PRIORITY,RESOURCE_ID,RESOURCE_TYPE,RESPONSE_CODE,TASKMGR_JOB_ID,TENANT_ID,UPDATED_AT,METADATA,SCHEDULED_AT) select generate_series(1,200000),'pod0.eu-west-0b','autobk_3b4d','2018-10-23 20:00:00.505','EXECUTE_SUCCESS','0316914de49845e281f80eb8b3be94db',null,'4daad27d-864b-47f0-bc0d-8f4e2abcc2ea',0,'457e9c2a-d161-4e5b-b803-51e8c041f395','volume',null,'40c89b186641d2cb0166a0ccf306068e','339f89f42c9e4248ac6f542b9fa913d7','2018-10-23 20:03:04.001',null,'2018-10-23 20:03:04.001';
```

