---
title: Frp之后的群晖获取真实来访ip
date: 2021-12-16 14:53:54
tags:
---

## 群晖环境

- 机型 DS3617xs
- 版本 6.1.7
- 开启 docker (镜像：nginx, stilleshan/frpc)
- 开启 ssh

### 1. 修改群晖 nginx 配置
备份 DSM.mustache
```
cp /usr/syno/share/nginx/DSM.mustache{,.bak}
```
在第一个 server 块加入以下代码
```
set_real_ip_from 172.17.0.0/16;
real_ip_header X-Real-IP;
```
重启 nginx
```
synoservice --restart nginx
```

### 2. 搭建 nginx 反代群晖
#### 2.1 在 `File Station` 中 `docker` 目录下新建 `nginx` 文件夹, 然后创建 `default.conf`
```
server {
    listen 443 ssl proxy_protocol;

    real_ip_header proxy_protocol;
    real_ip_recursive on;
    set_real_ip_from 172.17.0.0/16;
   
    ssl_certificate /etc/nginx/conf.d/cert;
    ssl_certificate_key /etc/nginx/conf.d/key;

    location / {
		proxy_set_header X-Real-IP       $proxy_protocol_addr;
    	proxy_set_header X-Forwarded-For $proxy_protocol_addr;
        proxy_pass http://172.17.0.1:5000;
    }
}
```
然后将 ssl 证书也导入 nginx 文件夹，并改名为 `cert` 和 `key`

#### 2.2 创建 nginx 容器
创建时选择 `高级设置 > 存储空间 > 添加文件夹` ，选择 `docker/nginx` 文件夹，`装载路径` 设置为 `/etc/nginx/conf.d` ， 然后正常创建即可

### 3. 创建 frpc 客户端
#### 3.1 在 `File Station` 中 docker 目录下新建 `frpc` 文件夹, 然后创建 `frpc.ini`
```
[common]
// 你的 frp 服务器配置

[dsm]
type = tcp
local_port = 443
local_ip = 172.17.0.2 // 上面创建的 nginx 的 docker 内网 ip
remote_port = 33333 // 看自己喜好选择端口即可
proxy_protocol_version = v2
```
#### 3.2 创建 frpc 容器
创建时选择 `高级设置 > 存储空间 > 添加文件`，选择 `docker/frpc/frpc.in`i 文件，`装载路径`设置为 `/frp/frpc.ini` ，然后正常创建即可

### 4. 访问 https://你的frp服务器地址:33333 如能正常访问，应该就说明配置好了


