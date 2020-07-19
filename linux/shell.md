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

### str

**判断字符串是否为空(zero)：**`[[ -z "" ]] && echo "yes" # yes`

**判断字符串是否非空：**`[[ -n "foo" ]] && echo "yes" # yes`

**判断字符串是否相等：**`[[ "foo" == "bar" ]] || echo "no" # no`

**判断字符串是否相等：**`[[ "foo" = "bar" ]] || echo "no" # no`

**判断字符串是否不相等：**`[[ "foo" != "bar" ]] && echo "yes" # yes`

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

### in操作

shell本身不支持`in`操作，可通过正则来实现：

```shell
if [[ ! $1 =~ ^(public_cloud|private_cloud|host)$ ]]; then
    echo "wrong deploy mode, only support public_cloud/private_cloud/host"
    exit 1
fi
```

## 常用

### 其他

**生成随机数：**`head /dev/urandom | tr -dc A-Za-z0-9 | head -c 5 ; echo ''`

**字符串替换：**`test=a,b,c;${test//,/_}`

**统计代码行：**`find src script  -name '*.h' -o -name '*.cpp' -o -name '*.sh' | xargs wc -l | sort -nr | head -n 10`

**生成ssh密钥：**`ssh-keygen -t rsa -b 4096 -C "shopppon@gmail.com"`

### 获取脚本路径

```shell
SCRIPT=$(readlink -f "$0")
SCRIPTPATH=$(dirname ${SCRIPT})
```

### 时间

#### 时间格式化

```shell
echo `date "+%Y-%m-%d_%H:%M:%S"`
date --help # more options
```

**将一个文件的时间属性拷贝到另外一个文件：**`touch -m -c -r blacklisted.certs cacerts`

## sed

**修改日志级别：**`sed -i 's/^\(verbose\|debug\)\s*=.*$/\1=False/' /etc/*/*.conf`

**删除匹配行：**`sed -i '/debug/d' /etc/*/*.conf`

**匹配到[]前面插入行：**`sed -i  '/^\[.*\]$/{x;p;x;}' *.conf`

**匹配到[]后面插入行：**`sed -i  '/^\[.*\]$/G;' *.conf`

**删除空白行：**` sed -i '/^$/d' *.conf`

**删除注释行：**`sed -i '/^#.*$/d' *.conf`

## awk

**过滤第三列大于0：**`awk '{$3>0 print $0}'`

**打印第二列：**`awk '{print $2}'`

### grep

**打印第二列：**`grep "hello" fale.dat || echo "hello" >> fake.data`

**输出前后N行：**`grep -C 3 "N: 100" /var/log/messages`

### cut

**打印所有的用户组名称：**`cut -d: -f1 /etc/group | sort`

### find

**查看多种类型文件：**

```shell
find . -type f -iname "*.log" -o -iname "*.gz" 
find . -type f \( -name "*.gz" -o -name "*.log" \)
find . -type f -regex '.*\(\.gz\|\.log\)'
find . -type f -regextype posix-extended -regex '.*.(log|gz)'
```

