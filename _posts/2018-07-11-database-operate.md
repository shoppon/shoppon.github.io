---
title: 常用数据库命令
layout: post
---

## 数据库指南

#### 常用命令

查询用户：```\du```

创建用户：```CREATE USER karbor_a WITH CREATEDB PASSWORD "passowrd";```

解锁用户：```alter user xxx login;```

修改表用户：```alter table xxx owner to xxx;```

#### 根据查询结果更新Text字段中的一部分内容

```sql
UPDATE checkpoint_item SET EXTEND_INFO = REPLACE(EXTEND_INFO, '"auto_trigger": false', '"auto_trigger": true') WHERE id='ae140853-2c58-4182-b874-c27a5bf59434';
```

#### 导出指定数据库
```sql
export LD_LIBRARY_PATH=/opt/gaussdb/app/lib/;/opt/gaussdb/app/bin/gs_dump karbor -W CloudService@123! -f /home/karbor.sql
```

#### 导入指定数据库
```sql
export LD_LIBRARY_PATH=/opt/gaussdb/app/lib/;/opt/gaussdb/app/bin/gsql -W CloudService@123! karbor --set ON_ERROR_STOP=on --single-transaction -f /opt/huawei/dj/etc/gaussdb/karbor.sql
```

#### 批量插入

```sql
INSERT INTO resources SELECT '2018-10-19 08:46:47.616534' AS CREATED_AT, '2018-10-19 08:46:47.616534' AS UPDATED_AT, NULL AS DELETED_AT, 'f' AS DELETED, i AS ID, '79ca9cb8-4078-4e64-8439-73cb0e1229c9' AS PLAN_ID, '8ec26d2d-4bd2-4fb2-bff7-5dd675883c31' AS RESOURCE_ID, 'OS::Nova::Server' AS RESOURCE_TYPE, 'ecs-xp' AS RESOURCE_NAME, '' AS RESOURCE_EXTRA_INFO FROM generate_series(80000, 80000) AS i;
```

```sql
delete from resources where id ~ '^\d{5,5}$';
```

#### 通过左外连接查询已删除Plans的副本
```sql
select checkpoint_item.id from checkpoint_item LEFT OUTER JOIN checkpoint ON checkpoint.id = checkpoint_item.checkpoint_id LEFT OUTER JOIN plans ON checkpoint.plan_id = plans.id where plans.deleted = 1
```

```
select checkpoint_item.id from checkpoint_item LEFT OUTER JOIN checkpoint ON checkpoint.id = checkpoint_item.checkpoint_id where checkpoint.plan_id in (select id from plans where plans.deleted = 1)
```

#### 修改执行时间

```sql
update TRIGGER_EXECUTIONS set EXECUTION_TIME='2018-10-27 09:30:00' where id in (select id from TRIGGER_EXECUTIONS where EXECUTION_TIME>'2018-10-27 09:00:00' and EXECUTION_TIME<'2018-10-27 10:00:00' limit 10);
```

#### 导入数据到文件
```sql
copy (select checkpoint_item.id from checkpoint_item) to '/home/gaussdba/tbl.csv' with csv header;
```

#### 查询空闲事务
```sql
select count(1) from pg_stat_activity where state='idle' and query!='';
```

#### 查看是否有锁
```sql
select QUERY,QUERY_START,STATE_CHANGE,WAITING,STATE from pg_stat_activity where WAITING='t';
```

#### 查看总连接数：
```sql
select count(1) from V$SESSION;
```

#### 查看karbor-a进程占的连接数：
```sql
select count(1) from V$SESSION where USERNAME='KARBOR_A';
```
