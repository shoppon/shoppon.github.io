---
title: "通过License控制界面特性展示"
categories: ["云计算"]
tags: ["license"]
date: 2021-01-21T20:00:00+08:00
typora-root-url: ../../post
---

# 背景

客户在某些场景下，需要使用**不同速率**的存储介质，配置不同的存储池；另外客户可能需要数据在不同存储集群间实现**物理隔离**，提升数据的安全性。

基于这两种背景，需要重新部署一套超融合ECS环境，**仅对外提供存储服务**。

因此，需要将界面上非存储相关的功能屏蔽，仅保留**自动化中心**、**平台升级**、**云监控服务**、**云基础设施**这四个入口。

# 上导航原理

当前界面的上导航菜单都是通过自定义的kubernetes资源 `navigation` 实现的，其中可见性控制行为共分为三种：

1. ECP基础云服务：通过`ark/ems-dashboard/templates/etc/_navigations.json.tpl`控制。
   1. 包括**云基础设施配置**、部门、项目、用户、访问密钥、配置管理、标签管理、可视化编排、编排部署、**自动化中心**、**平台升级**、云产品市场、已获取云产品、服务集成等14项。
2. Foundation云产品：通过`ark-horizon/chart/horizon/crtpl/xxx_navigation.yaml`控制。
   1. 包括计算(8)、网络(8)、存储(4)、资源编排(2)、备份与容灾(2)目录下所有服务，**云服务监控**服务等27项。
3. 独立云产品：通过`ark/{service}/templates/_navigation.yaml`控制。
   1. 包括定时器、容器、多区域管理等服务。

## ECP基础云服务

ECP基础云服务使用`_navigations.json.tpl`模板定义上导航菜单。

```
{
    "overview": {
        "category": "",
        "proId": "overview",
        "name": {
          "zh-cn": "概览",
          "en": "Overview"
        },
        "url": "/overview",
        "src": "",
        "order": 0,
        "icon": "overview",
        "available": true,
        "wizard": false
    },
    "vis_orc": {
        "category": "orchestration",
        "proId": "vis_orc",
        "name": {
            "zh-cn": "可视化编排",
            "en": "Visual Orchestration Service"
        },
        "url": "/ecos/stacks",
        "src": "",
        "icon": "vis_orc",
        "order": 0,
        "available": {{ .Values.ecs_default_menu.heat_enabled }},
        "wizard": false
    }
......
}
```

该模板会生成`status-operator-etc`的configmap

```shell
apiVersion: v1
kind: ConfigMap
metadata:
  name: status-operator-etc
data:
  aggregations.json: |
{{ tuple "etc/_aggregations.json.tpl" . | include "helm-toolkit.utils.template" | indent 4 }}
  categories.json: |
{{ tuple "etc/_categories.json.tpl" . | include "helm-toolkit.utils.template" | indent 4 }}
  navigations.json: |
{{ tuple "etc/_navigations.json.tpl" . | include "helm-toolkit.utils.template" | indent 4 }}
  nav_policy.json: |
{{ tuple "etc/_nav_policy.json.tpl" . | include "helm-toolkit.utils.template" | indent 4 }}
  wizards.json: |
{{ tuple "etc/_wizards.json.tpl" . | include "helm-toolkit.utils.template" | indent 4 }}
{{- end }}
```

`status-operator`容器会执行`status_operator.py`刷新ECP云服务的上导航列表。

`values.yaml`中定义的开头状态如下：

```shell
ecs_default_menu:
  domain: true
  neutron_l3: true
  eks_manage_enabled: true
  heat_enabled: true
  data_migration: false
  billing: false
  scheduler: true
  servicecatalog: true
```

## Foundation云产品

计算、存储、网络等foundation云产品通过创建`navigation`的方式注册上导航。

创建动作由`add-v5-menu-job`执行，其会从ark values中获取需要添加的导航列表，然后创建`navigation`。

```shell
command:
  - /add_v5_menu.sh
  {{- range $name, $mountPath := .Values.v5_menu_mount }}
  - {{ $name }}
  {{- end }}
```

创建逻辑如下：

```shell
components=$@
for component in ${components[@]}
do
    # enable dr navigation depending on whether the Dr switch is turned on in horizon.
    if [ $component = 'dr' ]; then
        DR_ENABLED=$(kubectl get cm -n openstack horizon-etc -o yaml | grep DR_ENABLED | awk '{print $3}')
        if [ $DR_ENABLED != 'True' ]; then
            continue
        fi
    fi
    create_cr_resource $component
done
```

需要创建的云服务列表从`values.yaml`文件中读取，其中`v5_menu_mount`的定义如下：

```shell
v5_menu_mount:
  murano:
    murano_navipolicy: murano_navipolicy.yaml
    murano_ecpstatus: murano_ecpstatus.yaml
    murano_navigation: murano_navigation.yaml
  aodh:
    aodh_navipolicy: aodh_navipolicy.yaml
    aodh_ecpstatus: aodh_ecpstatus.yaml
    aodh_navigation: aodh_navigation.yaml
  nova:
    nova_navipolicy: nova_navipolicy.yaml
    nova_ecpstatus: nova_ecpstatus.yaml
    nova_navigation: nova_navigation.yaml
    nova_wizard: nova_wizard.yaml
......
```

`add-v5-menu-job`将导航菜单创建出来后重启`ems-dashboard-status-operator`容器，将云产品的导航菜单合并到`servicecatalog` navigation中。

## 独立云产品

安装时会通过license关闭云产品市场功能，该场景暂不考虑。

# License原理

制作license时可以设置ark values，通过配置`ark/horizon`的`v5_menu_mount`属性和`ark/ems-dashboard`的`ecs_default_menu`属性可实现上导航菜单的可见性控制，其具体流程如下：

![license](/imgs/license.png)

更新license时，roller会从配置中解析需要升级的组件，其实就是`ark_update`下一级的组件名称，然后构建对应的ark values调用`helm upgrade`进行升级。

# 实现

## 方案一：在license中添加ark values

在制作ESS license时设置`v5_menu_mount`和`ecs_default_menu`，控制上导航是否可见。

具体需要的添加的ark values如下：

```shell
ark_update:
  horizon:
    values:
      v5_menu_mount:
        grafana:
          grafana_navipolicy: grafana_navipolicy.yaml
          grafana_ecpstatus: grafana_ecpstatus.yaml
          grafana_navigation: grafana_navigation.yaml
  ems-dashboard:
    values:
      ecs_default_menu:
        domain: false
        heat_enabled: false
        servicecatalog: false
```

该方案实现简单，不需要改动代码，但是需要在制作license时写入程序配置。同时，在导入全量特性license时，还需要在license中将上导航菜单配置中添加回来。另外，后期云服务增加可能也会牵扯到license制作参数变化。

该方案会导致上导航功能与license制作过程强耦合，不推荐。

## 方案二：roller配置license接口中修改ark values(最终方案)

license中仅标记是否仅支持ESS(参数名`use_ess_only`)，roller的config license接口中解析该参数，通过k8s client删除所有navigation，同时修改导航开关的ark_values并持久化到config cr，然后通过helm升级`horizon`和`ecp-dashboard`。

该方案使license制作和系统参数之间的不再耦合，但是后续新添加云服务依然可能需要修改代码。

后期导入全量特性license时，需要将`horizon`、`ems dashboard`的config cr中的上导航相关开关删除。

license中需要添加的配置参数为

```shell
ark_update:
  horizon:
    values:
      local_settings:
        use_ess_only: true
  ems-dashboard:
    values:
      local_settings:
        use_ess_only: true
```

## 方案三：ECP提供REST接口注册上导航菜单

在license中设置ark values会导致license制作麻烦，而且将系统内部参数完全暴露出来，方案上不是很合理。

ECP作为平台底座，应该通过接口的方式暴露该功能，与调用依赖方实现**解耦**，**隐藏**内部实现细节，方便升级和**演进**。（华为云的导航菜单就是通过REST接口注册的）

license中添加是否仅使用ESS的配置项，在license配置接口中调用ECP相关接口进行菜单的注册。

# 使用场景分析

## 场景一：安装时导入ESSOnly License

用户安装时使用ESSOnly License，安装后只显示ESS相关的4个菜单。

## 场景二：安装时导入试用License再导入ESSOnly License

用户安装时未导入license，这时全量菜单都会注册上来。导致ESSOnly License后需要将这些上导航菜单删除。

## 场景三：更新ESSOnly License为全量License

用户使用一段时间后，可能导入全量license，这里需要将所有上导航菜单注册。