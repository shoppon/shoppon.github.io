# 调试

## gdb使用

## 常用操作

**attach指定进程：**`gdb -p ${pid}`

**挂起所有进程：**`thread apply all backtrace`

**堆栈前进/后退：**`up/down`

### 生成core文件

1. ```ulimit -c unlimited```
2. ```echo "/tmp/core-%e-%p-%t" > /proc/sys/kernel/core_pattern```
3. 重启进程，core文件路径在```/tmp/core-xxx```。
4. 执行```gdb /path/to/bin /path/to/core```进行调试。

## strace：跟踪系统调用

**跟踪系统调用：**`strace -tt -f -s 128 -o /tmp/trace.log -p ${process}`

## 依赖

查看依赖`ldd oma`

查看符号`readelf -d oma`

## 参考

- [strace 跟踪进程中的系统调用](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/strace.html)
