---
title: 正则表达式
layout: post
---

### 基本用法

基本用法可以自行google。

### 零宽断言

零宽断言简而言之就是限制待匹配规则的前后必须是什么或者必须不是什么，可以简单概括如下：

| 规则         | 表达式 |
| ------------ | ------ |
| 前面必须是   | ?<=    |
| 后面必须 是  | ?=     |
| 前面必须不是 | ?<!    |
| 后面必须不是 | ?!     |

有一个文件内容如下：

```yaml
Name: demo
Version: "1.0"
Des: this is demo
```

需要将文件中的```Version```解析出来，正则可以概括为```前面必须是Version: "开头，版本号是数字，必须是“结尾。```

所以该正则可以表达为```grep -Po '(?<=Version:\s")[0-9.]+(?=")' $CONFIG_PATH```

### 高级正则样例

**FCS容器化etcd切dcm**

```find . -name *.tmpl|xargs -n1 -I{} bash -c 'sed -i "s/{{\s*if\s*exists\s*\(".*"\)\s*}}/{[if keyExists \1]}/g" {};sed -i "s/{{\s*\(else\|end\)\s*}}/{[\1]}/g" {};sed -i "s/{{\s*getv\s*\(".*"\)\s*}}/{[key \1]}/g" {}'``` 

sed不支持非贪婪模式，如果一行有多个匹配可能有问题，这种情况可以用```perl```替换。

```find . -name *.tmpl|xargs -n1 -I{} bash -c 'perl -pi -e "s|{{\s*if\s*exists\s*(".*?")\s*}}|{[if keyExists \1]}|g" {};perl -pi -e "s|{{\s*(else\|end)\s*}}|{[\1]}|g" {};perl -pi -e "s|{{\s*getv\s*(".*?")\s*}}|{[key \1]}|g" {}'```

#### 将代码中的配置转化成DCM配置

toggle.js文件中在很多开关，格式样例如下：

```json
{
    "IS_SUPPORTED_CSBS_DEC": true,
    "IS_SUPPORTED_CSBS_OP_LOG": false
}
```

需要将写死的开关变成Consul配置项，可用如下正则：

```sed -i 's#\(IS_SUPPORTED_CSBS.*\)".*$#\1": {[ keyOrDefault "DCM/REGION_ID_XX/SERVICE/ecsui/{{version}}/{{cluster_name}}/CONFIG/\L\1" "false" }],#g' toggle.js```

#### icaldendar小时格式检查

通过正则匹配```BYHOUR```后面不能出现非00~23的值。

```shell
p = re.compile(r"BYHOUR=(?!(([0-1][0-9]|2[0-3])(,([0-1][0-9]|2[0-3])){0,23})(;|\r))")
re.findall(p, "xxx;BYHOUR=22,12,23;xxx;BYHOUR=22\r;xxx;") # legal
re.findall(p, "xxx;BYHOUR=24;xxx;") # illegal
```

### 文本去重
假定有一个文件metadata.txt，其内容为如下：
```txt
blablabla\"key\": \"foo\"blablabla\"value\": \"bar9\"blablabla
blablabla\"key\": \"foo1\"blablabla\"value\": \"bar\"blablabla
blablabla\"key\": \"foo1\"blablabla\"value\": \"bar\"blablabla
blablabla\"key\": \"foo2\"blablabla\"value\": \"bar5\"blablabla
blablabla\"key\": \"foo3\"blablabla\"value\": \"bar7\"blablabla
```

现需要将key和value按照key去重后统计value的数量。

```shell
sed -r 's/^.*?\\"key\\":\s\\"(.*?)\\".*\\"value\\":\s\\"(.*?)\\".*$/\1 \2/g' metadata.txt |uniq|sort
```

