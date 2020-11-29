---
title: "家庭私有云网络架构"
tags: ["网络", "nginx", "harbor", "jumpserver"]
categories: ["云计算"]
date: 2020-10-21T09:06:30+08:00
draft: false
---

# 网络架构

光猫端口转发，软路由端口转发到ubuntu，ubuntu通过`nginx`反向代理。

# 配置

使用`nginx`作为反向代理，通过域名转发到不同后端，配置文件如下：

```shell
server
{
    listen 8080;
    server_name freenas.shoppon.xyz;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.5.27:80;
    }
    access_log logs/freenas_access.log;
}

server
{
    listen 8080;
    server_name blog.shoppon.xyz;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.5.58:31313;
    }
    access_log logs/blog_access.log;
}

server
{
    listen 8080;
    server_name openstack.shoppon.xyz;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.5.100:80;
    }
    access_log logs/openstack_access.log;
}

server
{
    listen 8080;
    server_name openwrt.shoppon.xyz;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.5.1:80;
    }
    access_log logs/openwrt_access.log;
}

server
{
    listen 8080;
    server_name harbor.shoppon.xyz;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass https://192.168.5.77:80;
    }
    access_log logs/harbor_access.log;
}

server
{
    listen 8080;
    server_name k8s.shoppon.xyz;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass https://192.168.5.54:6443;
    }
    access_log logs/k8s_access.log;
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream websocket {
    server 192.168.5.77:8088;
}

server
{
    listen 8080;
    server_name js.shoppon.xyz;
    location / {
        proxy_pass http://websocket;
        proxy_read_timeout 300s;
        proxy_send_timeout 300s;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection  $connection_upgrade;
    }
    access_log logs/js_access.log;
}

```

使用`proxy_set_header Host $host:$server_port;`可使被反向代理的网站跳转时依旧使用监听端口。

使用`proxy_set_header Host $host:$proxy_port;`可使被反向代理的网站跳转时依旧使用自身端口。

# Neutron

## 常用命令

### 绑定浮动IP
```shell
openstack floating ip set --fixed-ip-address 10.8.1.232 --port b7de6a74-d626-41d2-8245-f4bed029deec 192.168.5.24
```

# 参考

- [nginx 代理 websocket](https://juejin.im/post/6844903839749898247)
- [搭建nginx反向代理用做内网域名转发](http://www.ttlsa.com/nginx/use-nginx-proxy/)

