# 日志相关

## syslog

### 相关概念

**programname**：打印日志的程序名，目前Karbor用到了```karbor-api```、```karbor-apiControl```等几个。

**facility**：日志打印设备，默认是local1

**msg**：日志详细内容

**level**：日志级别

**format**：

#### python进程设置programname

python syslog handler(oslo_log.handlers.py)初始化时会取进程名，设置programname

```python
def __init__(self, facility=syslog.LOG_USER):
    # Do not use super() unless type(logging.Handler) is 'type'
    # (i.e. >= Python 2.7).
    logging.Handler.__init__(self)
    binary_name = _get_binary_name()
    syslog.openlog(binary_name, 0, facility)
```

#### rsyslog配置文件可以通过programename来设置过滤

```shell
# if filter all the msgs that contain keyword "token", all exception logs catched by faultwrap will be lost, so exclude keyword "auth_token".
if $programname startswith "karbor-" then {
    if ($msg contains_i "password")  then ~
    else {
        if $programname == "karbor-api" then {
            if $msg contains "OPTIONS / HTTP" then ~
            else {
                *.info ?karbor_api_file
            }
        }
    }
}
```

#### 通过命令行调用rsyslog打印

通过命令行方式打印日志可以使用-t参数指定programename，如：```logger -id -p local1.error -t {programnage} {level} {msg}```

## logrotate

**定时压缩日志：**`echo "*/5 * * * * dra /usr/sbin/logrotate -s /opt/dra/logs/status /opt/dra/conf/dra.logrotate" >>/etc/crontab`

*需要指定状态文件*

