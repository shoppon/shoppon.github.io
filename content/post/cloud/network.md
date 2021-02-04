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
upstream websocket_freenas {
    server 192.168.5.27;
}

server
{
    listen 8443;
    server_name freenas.shoppon.site;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.5.27;
    }
    location /websocket {
        proxy_pass http://websocket_freenas;
        proxy_read_timeout 300s;
        proxy_send_timeout 300s;

        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection  $connection_upgrade;
    }
    access_log logs/freenas_access.log;
}

server
{
    listen 8443;
    server_name blog.shoppon.site;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.5.58:31313;
    }
    access_log logs/blog_access.log;
}

upstream openstack {
        server 192.168.5.100:80;
}

server
{
    listen 8443;
    server_name openstack.shoppon.site;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://openstack;
    }
    access_log logs/openstack_access.log;
}

server
{
    listen 8443;
    server_name openwrt.shoppon.site;
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
    listen 8443;
    server_name harbor.shoppon.site;
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
    listen 8443;
    server_name k8s.shoppon.site;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass https://192.168.5.54:6443;
    }
    access_log logs/k8s_access.log;
}

server
{
    listen 8443;
    server_name manictime.shoppon.site;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.5.77:8086;
    }
    access_log logs/manictime_access.log;
}

server
{
    listen 8443;
    server_name share.shoppon.site;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.5.27:8005;
    }
    access_log logs/share_access.log;
}

map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream websocket_site {
    server 192.168.5.77:8088;
}

server
{
    listen 8443;
    server_name js.shoppon.site;
    location / {
        proxy_pass http://websocket_site;
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

server
{
    listen 8443;
    server_name ceph.shoppon.site;
    location / {
        proxy_redirect off;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://192.168.5.24:8080;
    }
    access_log logs/ceph_access.log;
}

```

使用`proxy_set_header Host $host:$server_port;`可使被反向代理的网站跳转时依旧使用监听端口。

使用`proxy_set_header Host $host:$proxy_port;`可使被反向代理的网站跳转时依旧使用自身端口。

### 代理httpd(openstack)

反向代理后面是`httpd`，比如`openstack`，需要修改`/etc/httpd/conf.d/15-horizon_vhost.conf`配置文件，添加```ServerAlias openstack.shoppon.site```配置。

### 代理Mysql

```shell
stream {
    upstream mysql {
        hash $remote_addr consistent;
        server 192.168.10.5:3306 max_fails=3 fail_timeout=30s;
    }
    server {
        listen 3306;
        proxy_connect_timeout 30s;
        proxy_timeout 600s;
        proxy_pass mysql;
    }
}
```

# Neutron

## 常用命令

### 绑定浮动IP
```shell
openstack floating ip set --fixed-ip-address 10.8.1.232 --port b7de6a74-d626-41d2-8245-f4bed029deec 192.168.5.24
```

# 网络配置

## 使用dnat端口转发

```shell
iptables -t nat -I PREROUTING -d 10.10.1.4 -p tcp -m tcp --dport 6443  -j DNAT --to-destination 192.168.4.43:6443
```

# 参考

- [nginx 代理 websocket](https://juejin.im/post/6844903839749898247)
- [搭建nginx反向代理用做内网域名转发](http://www.ttlsa.com/nginx/use-nginx-proxy/)

