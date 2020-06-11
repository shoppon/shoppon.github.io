# cgroup资源限制

## 简介

`cgroup`（control group）是linux系统内核提供的一种资源限制机制。`docker`便是使用`cgroup`来进行资源限制。

## 使用

### 创建cgroup

`cgroup`以文件形式提供接口，直接创建对应的文件即可：

```shell
cgroup_cpu_path=/sys/fs/cgroup/cpu/dra
mkdir ${cgroup_cpu_path} | :
echo 10000 >${cgroup_cpu_path}/cpu.cfs_period_us
echo 1000 >${cgroup_cpu_path}/cpu.cfs_quota_us

cgroup_mem_path=/sys/fs/cgroup/memory/dra
mkdir ${cgroup_mem_path} | :
# limit 500MB
echo 524288000 >${cgroup_mem_path}/memory.limit_in_bytes
```

**cpu.cfs_period_us**: 完全公平调度器的调整时间配额的周期。

**cpu.cfs_quota_us**: 完全公平调度器的周期当中可以占用的时间。

设置为1000/1000则会限制占用10%。

### 使用cgropu启动进程

在使用`systemd`管理进程的场景中，只需要在`dra.sysconfg`配置文件中设置`CGROUP_DAEMON`为所需使用的cgroup名称即可，具体原理为：

```shell
# if they set CGROUP_DAEMON in /etc/sysconfig/foo, honor it
if [ -n "${CGROUP_DAEMON}" ]; then
    if [ ! -x /bin/cgexec ]; then
        echo -n "Cgroups not installed"
        warning
        echo
    else
        cgroup="/bin/cgexec"
        for i in $CGROUP_DAEMON; do
            cgroup="$cgroup -g $i"
        done
    fi
fi

# And start it up.
if [ -z "$user" ]; then
    $cgroup $nice /bin/bash -c "$corelimit >/dev/null 2>&1 ; $*"
else
    $cgroup $nice runuser -s /bin/bash $user -c "$corelimit >/dev/null 2>&1 ; $*"
fi

```

### 查询进程使用的cgroup

可通过进程id查询其所使用的cgroup：

```shell
cat /proc/104980/cgroup
12:memory:/system.slice/system-hostos.slice/dra.service
11:blkio:/system.slice/system-hostos.slice/dra.service
10:perf_event:/
9:pids:/system.slice/system-hostos.slice/dra.service
8:freezer:/system.slice/system-hostos.slice/dra.service
7:files:/
6:devices:/system.slice/system-hostos.slice/dra.service
5:hugetlb:/
4:net_prio,net_cls:/
3:cpuset:/
2:cpuacct,cpu:/system.slice/system-hostos.slice/dra.service
1:name=systemd:/system.slice/system-hostos.slice/dra.service
```

## 效果
