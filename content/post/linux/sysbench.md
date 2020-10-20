# 基准测试

## 编译

`sysben`工具地址为[akopytov/sysbench)](https://github.com/akopytov/sysbench)，按照官方指导编译即可。

## 使用

### CPU测试

**CPU基准测试：**`./sysbench cpu --cpu-max-prime=40000 run`

### 内存测试

**内存基准测试：**`./sysbench memory --memory-block-size=8k --memory-total-size=80G run`

### 磁盘测试

通过创建文件随机读写方式测试磁盘性能：

```shell
./sysbench fileio --file-num=16 --file-total-size=128M --file-test-mode=rndrw prepare
./sysbench fileio --file-num=16 --file-total-size=128M --file-test-mode=rndrw run
./sysbench fileio --file-num=16 --file-total-size=128M --file-test-mode=rndrw cleanup
```

测试结果如下：

```shell
sysbench 1.0.20-ebf1c90 (using bundled LuaJIT 2.1.0-beta2)

Running the test with following options:
Number of threads: 1
Initializing random number generator from current time


Extra file open flags: (none)
128 files, 1MiB each
128MiB total file size
Block size 16KiB
Number of IO requests: 0
Read/Write ratio for combined random IO test: 1.50
Periodic FSYNC enabled, calling fsync() each 100 requests.
Calling fsync() at the end of test, Enabled.
Using synchronous I/O mode
Doing random r/w test
Initializing worker threads...

Threads started!


File operations:
    reads/s:                      203.31
    writes/s:                     135.54
    fsyncs/s:                     433.82

Throughput:
    read, MiB/s:                  3.18
    written, MiB/s:               2.12

General statistics:
    total time:                          10.9160s
    total number of events:              8309

Latency (ms):
         min:                                    0.00
         avg:                                    1.23
         max:                                  300.41
         95th percentile:                        7.43
         sum:                                10209.83

Threads fairness:
    events (avg/stddev):           8309.0000/0.00
    execution time (avg/stddev):   10.2098/0.00
```
