# Shell编程

## 基础

- 在文件开头声明使用的解释器：`#!/bin/bash`
- 设置不允许出错：`set -e`
- 设置打印输出：`set -x`

## 参数

通过`getopts`获取参数

```shell
while getopts ':u' OPT; do
    case $OPT in
    u)
        echo "Upgrade."
        ;;
    ?)
        echo "Usage: -u upgrade"
        exit 1
        ;;
    esac
done
shift $(($OPTIND - 1))
```

## 数据结构

### map

使用`declare`声明

```shell
declare -A UUID_MAP=(["ra"]="100"
    ["soma"]="200"
    ["toma"]="300")
```

## 控制

### for循环

```shell
for d in ra soma toma; do
    start_docker ${d}
done
```

### if

```shell
if [ -d ${OMA_PATH} ]; then
    ls ${OMA_PATH} | grep -v logs | xargs -I{} rm -rf ${OMA_PATH}/{}
fi
```

## 常用

### 其他

**生成随机数：**`head /dev/urandom | tr -dc A-Za-z0-9 | head -c 5 ; echo ''`

**字符串替换：**`test=a,b,c;${test//,/_}`

### 获取脚本路径

```shell
SCRIPT=$(readlink -f "$0")
SCRIPTPATH=$(dirname ${SCRIPT})
```

## sed

**修改日志级别：**`sed -i 's/^\(verbose\|debug\)\s*=.*$/\1=False/' /etc/*/*.conf`

**删除匹配行：**`sed -i '/debug/d' /etc/*/*.conf`

## awk

**过滤第三列大于0：**`awk '{$3>0 print $0}'`

**打印第二列：**`awk '{print $2}'`

### grep

**打印第二列：**`grep "hello" fale.dat || echo "hello" >> fake.data`

### cut

**打印所有的用户组名称：**`cut -d: -f1 /etc/group | sort`

