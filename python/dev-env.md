---
title: 开发环境搭建
layout: post
---

# 开发环境搭建

#### python环境设置

1. 安装pip，具体方式google
2. 安装[VCForPython27](https://www.microsoft.com/en-us/download/details.aspx?id=44266)，用于编译c语言
3. 由于网络原因无法从外网pip源下载三方包，需要将pip源设置为华为内源。
  - 在个人目录（C:\Users\x00250203\pip）创建pip目录，添加pip.ini文件
  - 将文件内容设置为：
```
[global]
index-url=http://rnd-mirrors.huawei.com/pypi/simple/
[install]
trusted-host=rnd-mirrors.huawei.com
```
4. 从requirments安装三方包，进入karbor/src/karbor， 编辑requirements.txt，由于内部源软件缺少新版本，需要降低版本需求或直接注释掉版本需求，然后执行```pip install -r requirments.txt```
5. mock kmc，进入python的site-packages目录，创建kmc目录和__init__.py将，将[kmc.py](http://code.huawei.com/csbs/dev/blob/master/dev/kmc/kmc.py)拷贝到kmc目录
6. 如果运行过程中报```cannot import module xxx```，直接使用```pip install xxx```安装即可，**特殊说明：openstack client xxxclient需要安装python-xxxclient**。

#### pycharm设置 ####
- 换行符设置为LF(Linux格式)：```File```-->```Keymap```-->```Main menu```-->```File```-->```Line Separators```-->```LF```
- 编码格式设置为UTF-8，python头文件添加```code: utf-8```
- 设置单行文件最大79字符（PEP8规范）：```File```-->```Setting```-->```Editor```-->```Code Style```-->```Right margin```
- pycharm导入工程强烈建议不要把整个代码目录导入进来，导入karbor和kangaroo两个工程即可，并设置依赖。

#### git设置 ####
- 设置不把LF转化成CRLF（默认会转）：```git config --global core.autocrlf false```
- 设置ignore文件：```git config --global core.excludesfile '~/.gitignore'```
- 设置中文不乱码：```git config --global core.quotepath false```

#### rabbitmq搭建
- rabbitmq是erlang编写，首先需要下载安装[erlang](http://www.erlang.org/downloads)，[codeclub下载](http://code.huawei.com/csbs/dev/tree/master/tools/erlang)
- 安装windows版[rabbitmq](https://www.rabbitmq.com/install-windows-manual.html)，[codeclub下载](http://code.huawei.com/csbs/dev/tree/master/tools/Rabbitmq)
- 双击```rabbitmq-service.bat```运行rabbitmq

#### mysql环境搭建

由于原生postgres不支持bool值和integer值转换，导致无法使用postgres数据库进行调试。[mysql下载](https://dev.mysql.com/downloads/mysql/) [codeclub下载](http://code.huawei.com/csbs/dev/tree/master/tools/Mysql)

安装Python的mysql驱动```pip install mysql-connector==2.1.6```（高版本有BUG）

设置外键不检查：```SET GLOBAL FOREIGN_KEY_CHECKS=0;```

#### openssl搭建

token的解密会用到openssl相关命令，因为需要在本地安装openssl，[下载地址](http://code.huawei.com/csbs/dev/tree/master/tools/openssl)

#### python运行设置

openstack本身是运行在linux上的，如果需要运行在windows上需要做如下适配：

1. 修改```site-packages\keystonemiddleware\auth_token\__init__.py```文件里的```_cms_verify```函数里的```verify```函数如下：

   ```python
   from kangaroo.tests.debug import token
   verified_data = token.verify(data)
   if not verified_data:
       raise ksc_exceptions.CertificateConfigError("fake")
   return verified_data
   ```

2. 修改```site-packages\keystonemiddleware\auth_token\_signing_dir.py```文件里的```_verify_signing_dir```函数，在第一句```if os.path.isdir(self._directory_name): return 0```（新增），否则修改文件失败。

3. 修改```site-packages\eventlet\greenio\base.py```文件里的 ```set_nonblocking```函数, 在```except AttributeError```的注释后加return

4. 修改```site-packages\keystonemiddleware\auth_token\__init__.py```文件里的```_validate_offline```函数，给语句```self._revocations.check(token_hashes)```之前加判断条件```if self._check_revocations_for_cached:```

5. 修改```site-packages\keystoneauth1\identity\base.py```文件里的get_endpoint_data函数，把```if interface is plugin.AUTH_INTERFACE:```修改为```if True or interface is plugin.AUTH_INTERFACE:```

6. 修改```kangaroo\src\kangaroo\etc\debug\karbor.conf```文件的```[keystone_authtoken] insecure = true```；

7. 修改```kangaroo\src\kangaroo\etc\debug\api_paste.ini```文件里```[filter:authtoken]   signing_dir = D:\runtime\keystone-signing\```

8. 修改```karbor\src\karbor\karbor\utils.py```里的```find_config```函数，在```possible_locations```数组里加一条```os.path.join("etc\\debug", config_path)```

#### 本地运行karbor(openstack)

如果想深入研究openstack wsgi框架，oslo相关类库，本地调试是最好的手段，具体步骤如下。

1. 生成entry_point
    拷贝```/kangaroo/src/kangaroo/build```下文件到Karbor根目录```karbor/src/karbor```中，分别进入到```karbor/src/karbor```和```/kangaroo/src/kangaroo```根目录，执行```python setup.py egg_info```生成Karbor和Kangaroo的entry_point。

2. 拷贝```/kangaroo/src/kangaroo/karbor_extend```目录下文件到karbor目录```karbor\src\karbor\karbor```中，覆盖karbor原生代码

3. 应用Karbor补丁
    利用git bash进入CSBS根目录，执行```git apply kangaroo/src/kangaroo/patches/*```

4. 生成Karbor数据库

   本地登录mysql，新建数据库，取名为karbor

   编辑```kangaroo/src/kangaroo/etc/debug/karbor.conf```，修改connection中的用户名密码为本地mysql的用户名密码

6. 本地添加host，具体路径```C:\Windows\System32\drivers\etc\hosts```，添加域名解析```iam-cache-proxy.domainname.com```为```100.130.41.73```（IP地址可能需要根据实际情况修改）
    注意：如果IE配置了http代理，需要将该域名排除在外

7. 连接VPN
8. 启动karbor相关进程
  - 右键```kangaroo/tests/debug/api.py```文件，选择```run api.py```。
  - 设置运行参数为```--config-file=D:\CSBS_code\CSBS\kangaroo\src\kangaroo\etc\debug\karbor.conf```。（需根据本地路径调整）
  - 设置工作路径为```D:\CSBS_code\CSBS\kangaroo\src\kangaroo```。（需根据本地路径调整）
  - 对```kangaroo/tests/debug/protection.py```和```kangaroo/tests/debug/operationengine.py```执行相同操作。

### 通过postman调用本地接口
1. 下载postman(自行google)，导入[postman配置文件](http://code.huawei.com/csbs/dev/tree/master/tools/PostMan)
2. Postman中点击File->Settings打开设置，在General页下将SSL证书验证关闭，在Proxy页下将代理关闭
4. 设置环境为Local，默认已设置鉴权相关```IAM_DOMAIN```、```PROJECT_ID```、```USERNAME```、```DOMAIN_NAME```等环境变量，可根据需要修改。
5. 通过keystone中```IAM获取租户Token```接口获取用户token，并且设置到环境变量```TOKEN```中。
6. 通过csbs接口调用本地接口进行调试。

### 常见问题
*  运行api时报kmc不存在：
>  解决方法：将kmc压缩包中的kmc目录解压到python的Lib\site-packages目录中，[kmc.rar](/uploads/782491278b2742d2f6748997629ab830/kmc.rar)

* 支持时报三方包不存在：
>  解决方法：pip install python-XXXmoduleName。  [pip的使用方法](http://code.huawei.com/csbs/dev/wikis/%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA)

### CI LLT环境搭建
* 安装pbr，```pip install pbr```
* 安装nosetest所需三方包```pip install --upgrade flake8 flake8-junit-report hacking pep8-naming testtools nose coverage mock nose-cov nose_xunitmp```
* 安装Karbor运行时所需要的三方包
  - 解压CSBS软件安装包
  - 进入```CSBS/repo/python```目录，执行```rpm -ivh *.rpm --force --nodeps```安装所有rpm包
  - 进入```CSBS/repo```目录，执行```rpm -ivh *.rpm --force --nodeps```安装所有rpm包
  - 进入```/installfiles/repo```目录，执行```rpm -ivh *.rpm --force --nodeps```
  - 在```/usr/lib/python2.7/site-pakcages```目录新建```kmc```目录，将kmc mock
* 将kangaroo目录加入到python环境变量中```echo `pwd` >  /usr/lib/python2.7/site-packages/kangaroo.pth```
* 安装obs sdk
  - 将platform目录下的OBS SDK拷到一个自定义目录然后解压，进入src目录执行 ```python setup.py install``` 
* 连接openvpn：```openvpn --cd /etc/openvpn/l2/ --config /etc/openvpn/l2/client.ovpn daemon```

### Jenkins环境搭建
由于系统盘空间较小，需要自定义jenkins目录，执行```export CATALINA_OPTS="-DJENKINS_HOME=/home/.jenkins/ -Xmx512m"```，然后重启jenkins，执行```/usr/local/src/apache-tomcat-7.0.77/bin/catalina.sh start```

### Jenkins CI与Codeclub集成
* [Jenkis插件配置](http://3ms.huawei.com/km/blogs/details/2168171?l=zh-cn)

#### CI挂载CIFS共享，自动推包

- 安装samba和cifs工具包```yum install cifs-utils```、```yum install samba-client```
- 使用mount命令挂载```mount.cifs //10.156.17.188/CTU_CI_NAS/Storage_DP_OceanStor_BCManager_eBackup/karbor /opt/archive/  -o user=xxx```

