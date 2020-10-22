---
title: "VSCode remote使用方法"
categories: ["vscode", "效率", "开发工具"]
tags: ["vscode", "remote"]
date: 2019-12-30T19:35:08+08:00
---

# 前言

**按：** vscode是宇宙第一编辑器，不接受任何反驳。

vscode是微软公司推出的一款开源轻量级编辑器，支持各种主流语言，拥有强大丰富的插件机制。其中最令人心动的是其官方```remote development```插件，此插件一出谁与争锋！

vscode remote简单说就是在本地直接进行远程开发，源代码、编辑器、编译环境、调试环境全部是远程的测试环境，本地vscode只是起访问客户端的作用（类似于TC盒子的作用）。使用该工具之后，本地甚至都不需要源代码，而且可以使用linux上各种丰富、强大的命令行(grep、awk)，以及管理测试环境上的各种docker容器。

# 安装方法

vscode remote的方法非常简单，首先你要拥有一台与本地windows办公环境能连通的linux调测云（我使用的是ubuntu 16.04），具体配置步骤如下：

1. 在windows办公环境安装[vscode](https://code.visualstudio.com/)。

2. 安装完成后在插件市场中搜索```remote development```并安装。

3. 配置调测云环境的```~/.wgetrc```文件，第一次登陆时会自动下载vscode linux安装，具体配置如下：

   ```shell
   check_certificate=off
   https_proxy=http://{your_name}:{your_password}@proxy.xxx.com:8080
   http_proxy=http://{your_name}:{your_password}@proxy.xxx.com:8080
   use_proxy=on
   ```

   如果密码中含有特殊字符，则需要转义一下，点击```F12```打开chrome控制台，输入```encodeURIComponent('@')```就可以得到转义后的字符。

4. 如果是```Euler```操作系统，还需要将```/etc/ssh/sshd_config```中的```AllowTCPForwarding```和```PermitTunnel```设置为```yes```，否则无法安装插件。

5. 在本地vscode中按```ctrl+shift+p```，输入```remote-ssh Connect to host```执行，然后按提示输入远程调测云的账号、密码。

6. 等待远程vscode的安装完成。

# 推荐插件

vscode拥有很多强大的扩展插件，我个人在使用的如下这些：

- docker：管理容器镜像、执行命令。
- clang-format：代码格式化。
- gitlen：git辅助工具。
- tabnine：智能补全工具。
- shell-format：shell格式化工具。
- Doxygen Documentation Generator：生成文档化注释。
- vim：vim模式。
- Code Runner：一键执行代码。
- Todo Tree：代码TODO标签管理。
- One Dark Pro：个人比较喜欢的一款主题。

# 远程调试

vscode不仅仅在编辑方面比较出色，其还拥有强大的调试能力，以OMA为例，其可以交互式的调用```gdb```进行调试，以非常友好的方式在编辑器中执行```gdb```的各种操作，比如查看源代码、运行跳转、查看变量值、断点管理等。具体使用方法如下：

1. 在工程的根目录创建```.vscode```目录，添加```launch.json```文件，内容如下：

   ```shell
   {
     "version": "0.2.0",
     "configurations": [
       {
         "type": "cppdbg",
         "request": "launch",
         "name": "sOMA",
         "program": "/home/x00250203/code/DRA/src/build/datapath/oma/oma",
         "stopAtEntry": true,
         "args": [
           "OMA",
           "-o",
           "/etc/oma/soma.conf",
           "-s",
           "/etc/oma/soma_sessions/"
         ],
         "cwd": "${workspaceRoot}"
       }
     ]
   }
   ```

2. 将上述文件内容的路径修改为正确的环境路径。

3. 在调试窗口中选择```sOMA```，点击开始调试。
