# 灾备的四种技术路线

众所周知，灾备共有4种技术路线：

**存储层：**通过存储系统内建的固件或操作系统、IP网络或光纤通道等传输介质连结，将数据以同步或异步的方式复制到远端，从而实现生产数据的灾难保护。采用基于存储数据复制技术建设容灾方案的特点主要是对**网络连接**及**硬件**的要求较高。

**虚拟化层：**存储虚拟化是在存储设备中形成的存储资源透明抽象层。即存储虚拟化是服务器与存储间的一个抽象层，它是物理存储的逻辑表示方法。存储网关通过对于进入的IO数据流提供各类数据存储服务，大幅提升了在服务器或者存储层面难以达到的灵活性、多样性、异构化等多种存储服务能力。

**主机层：**基于主机的数据复制是通过**磁盘卷**的镜像或复制进行的，业务进行在主机的卷管理器这一层，对硬件设备尤其是存储设备的限制小，利用生产中心和备份中心的主机系统通过IP网络建立数据传输通道，数据传输比较可靠，效率也相对较高；通过主机数据管理软件实现数据的远程复制，当主数据中心的数据遭到破坏时，可以随时从备份中心恢复应用或从备份中心恢复数据。

**应用层：**数据库级容灾是建立在数据库基础上，它将源数据库中的数据库通过逻辑的方式在异地建立一个同样的数据库，并且实时更新，当主数据库发生灾难时可及时接管业务系统，达到容灾的目的。采用数据库层面的数据复制技术进行灾备建设具有投资少、无需增加额外硬件设备、可完全支持异构环境的复制等优点。基于数据库的数据复制是对数据库记录级别、表级别容灾高可用的基础技术。



以5星为满分，分数越**高**代表在该对比项上**优势**越明显，各种技术路线的对比如下：

| 对应项     | 存储层 | 虚拟化层 | 主机层 | 应用层 |
| ---------- | ------ | -------- | ------ | ------ |
| 实时性     | ★★★★★  | ★★★☆☆    | ★★★☆☆  | ★★★☆☆  |
| 一致性     | ★☆☆☆☆  | ★★☆☆☆    | ★★★☆☆  | ★★★★★  |
| 存储兼容性 | ★☆☆☆☆  | ★★★☆☆    | ★★★★★  | ★★★★★  |
| 系统兼容性 | ★★★★★  | ★★★★☆    | ★☆☆☆☆  | ★★★★★  |
| 应用兼容性 | ★★★★★  | ★★★★☆    | ★★★☆☆  | ★☆☆☆☆  |
| 业务影响   | ★★★★★  | ★★★★☆    | ★★☆☆☆  | ★☆☆☆☆  |
| 网络成本   | ★☆☆☆☆  | ★★★★☆    | ★★★☆☆  | ★★☆☆☆  |
| 数据缩减率 | ★★★☆☆  | ★★☆☆☆    | ★☆☆☆☆  | ★★★☆☆  |
| 安全性     | ★★★★☆  | ★★★☆☆    | ★☆☆☆☆  | ★☆☆☆☆  |
| RPO        | ★★★★★  | ★★★☆☆    | ★★★★☆  | ★★☆☆☆  |
| RTO        | ★☆☆☆☆  | ★★★☆☆    | ★★☆☆☆  | ★★★★☆  |
| TCO        | ★☆☆☆☆  | ★★☆☆☆    | ★★★☆☆  | ★★★★☆  |
| 技术复杂度 | ☆☆☆☆☆  | ★★☆☆☆    | ☆☆☆☆☆  | ☆☆☆☆☆  |



## 缓存的N种技术路线

与灾备的4种技术路线类似，缓存技术大概也有几种：

**客户端：**客户端软件(浏览器/App)本身的缓存功能。

**网络层：**主要有`CDN`技术。

**应用层：**主要有`redis`、`memcached`等。

**存储层：**存储设备的`SmartCache`等。
