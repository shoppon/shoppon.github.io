# Linux常用命令及其用法举例

### sed

**删除匹配行**

```shell
sed -i '/127.0.0.1/d' fake.dat
# mac下需要添加空格
sed -i '' '/127.0.0.1/d' fake.dat
```

### awk

**杀掉指定名称进程**

```shell
ps -ef|grep {process}|awk '{print $2}'|xargs kill -9
```

### grep

**如果不存在则插入一行**

```shell
grep "hello" fale.dat || echo "hello" >> fake.data
```

