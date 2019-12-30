# Ansible常用操作

## 原理

```ansible```是一款业界出名的**运维工具**，可将linux相关操作封装成```playbook```，将主机信息通过```inventory```进行管理。

## 安装

```ansible```是基于python开发的，执行```pip install ansible```即可。

## inventory样例

```shell
[fsb]
172.18.12.10

[fsb:vars]
ansible_ssh_user=dsware
ansible_ssh_pass=Huawei@CLOUD8
ansible_become=yes
ansible_become_method=su
ansible_become_user=root
ansible_become_pass=Huawei@CLOUD8!
```

## playbook样例

```shell
- hosts: fsb
  remote_user: root
  tasks:
    - name: clean
      shell: rm -rf /home/dra*
    - name: scp package
      copy:
        src: "{{item}}"
        dest: /home/dra.tar.gz
      with_fileglob: "/home/x00250203/code/DRA/build/output/dra-*.tar.gz"
    - name: untar package
      unarchive:
        src: /home/dra.tar.gz
        dest: /home/
        remote_src: yes
    - name: install
      shell: cd /home/dra-*/script/ && bash install.sh
    - name: restart dgw
      shell: ps -ef|grep dsware_dgw|grep -v grep|awk '{print $2}'|xargs kill -9

```

## 常用ansible命令

**运行指定play-book：**```ansible-playbook -i /home/x00250203/env/fsb/hosts install_dra.yaml```
