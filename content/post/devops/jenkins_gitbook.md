---
title: "Gitlab集成jenkins自动编译gitbook"
categories: ["devops"]
tags: [""]
date: 2019-12-30T20:00:43+08:00
---

# Gitlab集成jenkins自动编译gitbook

### 安装nodejs

1. 执行```echo "registry = http://mirror/npm" >> ~/.npmrc``` ，设置npm源为内源。

### 安装gitbook

1. 执行```npm install -g gitbook-cli```安装gitbook

### Jenkins配置

1. 在插件市场中搜索```gitlab```插件并安装gitlab。

2. 配置```jenkins```用户的git信息，执行```su jenkins```进入jenkins用户，运行```git config --global user.name "jenkins"```和```git config --global user.email "jenkins@x00250203"```。

3. 创建jenkins工程，执行以下脚本：

   ```
   cd $WORKSPACE
   gitbook build . public
   git checkout origin/ipages
   ls |grep -v public|xargs rm -rf && mv public/* . && rm public -rf
   git add -A
   git commit -m $GIT_COMMIT
   ```

4. 在```gitlab```插件中点击```Advanced```-->```screct token```-->```generate```，生成token。

5. 添加```git publisher```构建后动作，远程分支设置为```ipages```。

### Gitlab配置

在gitlab设置中添加webhook，url选择创建的工程路径，token选择上述步骤生成的token。
