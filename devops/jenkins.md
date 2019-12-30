# Jenkins安装、配置

### 安装

1. 执行 ```yum install java-1.8.0-openjdk```安装jdk8。
2. 执行```wget http://pkg.jenkins.io/redhat-stable/jenkins-2.190.3-1.1.noarch.rpm```下载软件包。
3. 执行```rpm -ivh jenkins*.rpm```安装。

### 系统配置

- 将```jenkins```用户加入到```docker```组：```usermod -aG docker jenkins```，否则无法使用docker。

### 配置存储

强烈建议为jenkins使用独立的数据存储，推荐使用```lvm```的```logic volume```，这样系统崩溃、重装时数据不会丢失。

jenkins配置路径为```/etc/sysconfig/jenkins```，可自定义数据目录和监控端口等，

```powershell
JENKINS_HOME="/home/jenkins/"
JENKINS_USER="root"
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Dhudson.model.DownloadService.noSignatureCheck=true"
```

配置java启动参数，否则可能无法获取插件列表，报证书校验错误。

### 添加权限

jenkins默认使用```jenkins```用户执行任务，有可能遇到权限不足的情况，索性使用root权限。编辑```/etc/sudoers```，添加```jenkins ALL=(ALL)       ALL```。

