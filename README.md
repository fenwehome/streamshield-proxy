# StreamShield Proxy:  HLS MPD流媒体播放多源代理聚合工具

## 简介

StreamShield Proxy 是一个强大的多源流媒体聚合代理工具，旨在解决因 IP 限制而无法直接播放各种流媒体内容的问题。它支持多个VPS部署，形成CDN网络，为用户提供流畅的流媒体播放体验。

## 核心功能

1.多源聚合：支持导入在线url，本地文件导入多个流媒体源

2.智能代理：自动处理需要代理的内容

3.内容定制：支持移除tv-logo、修改group-title等自定义操作

4.安全加固：集成安全token机制

5.灵活配置：通过环境变量实现灵活配置

6.高效缓存：使用LRU缓存优化性能

7.跨平台支持：支持arm64和amd64架构

## 发展现状

1.全新的多源聚合机制

2.优化的缓存系统，提高响应速度

3.增强的错误处理和日志记录

4.支持对特定源的内容进行自定义修改

5.改进的SSL证书错误处理

💖 **特别鸣谢：** 衷心感谢 AKTV 的作者！他们发布的程序让大家能更方便地收看海外电视台。本项目已将aktv m3u设置为默认在线源。

## 环境变量配置

所有thetv pixman的视频源都能以文件形式写在文件内方便持久化保存。先成功运行一次docker，本地文件夹就会生成所有预先生成的子文件夹和文件。

通过持久化 -v /your_local_path/:/app/config  挂载

/app/config/

* **proxy_hosts/**
    * `default.txt` - 默认代理 hosts 列表 (默认创建，每次升级我会覆盖)
    * `user_defined.txt` - 用户自定义代理 hosts 列表 (预创建，用户编辑，升级不会覆盖)

* **remote_m3u/**
    * **no_proxy/**
        * `default_sources.txt` - 默认非代理 M3U 源列表 (默认创建，每次升级我会覆盖)
        * `sources.txt` - 用户自定义非代理 M3U 源列表 (预创建，用户编辑，升级不会覆盖)
    * **proxy_needed/**
        * `sources.txt` - 用户自定义代理 M3U 源列表 (预创建，用户编辑，升级不会覆盖)

* **local_m3u/**
    * **no_proxy/**
        * `user_m3u.m3u` - 用户本地非代理 M3U 文件 (用户手动添加，文件名随便取，后缀支持m3u和txt)
    * **proxy_needed/**
        * `user_m3u.m3u` - 用户本地代理 M3U 文件 (用户手动添加，文件名随便取，后缀支持m3u和txt)

* **generated/**
    * `all.m3u` - 聚合的 M3U 文件 (程序自动生成)


## 终端播放器

在 Android 环境下，您需要使用 [https://github.com/FongMi/Release/tree/fongmi/apk/release](https://github.com/FongMi/Release/tree/fongmi/apk/release) 支持 mpd 加密解码播放。

ios下用aptv订阅现已支持。不支持mpd加密频道。在原先订阅链接后加上all.m3u便可。

Telegram channel https://t.me/tvb_ys

# Docker Compose部署指南

一键部署只支持在美国加拿大vps部署正常收看thetv，。如想分别部署thetv和播放在不同vps上 可以在美国vps上安装thetv docker 然后自行修改env和yml里的地址和端口号

## 前置条件

- Docker 和 Docker Compose安装完成

## 克隆本仓库
```bash
git clone https://github.com/pppyyyccc/streamshield-proxy.git

cd streamshield-proxy
```

## 配置

编辑 .env 文件，设置您的个人配置：
```bash
  nano .env
```
## 主要配置项：

- VPS_HOST: 设置为您的服务器 IP 或域名
- SECURITY_TOKEN: 设置一个安全的访问令牌
- MYTVSUPER_TOKEN: 如果使用 MyTVSuper，填入您的令牌

主动修改端口号，根据需要调整其他配置项。

根据需要调整其他配置项。

## 使用
启动服务 在项目目录中运行：
```bash
  docker-compose up -d
 ```

进入/config添加修改链接和文件m3u

```bash
docker-compose restart streamshield-proxy
 ```
## 访问
在ok影视终端播放器内-设置-直播 填入以下格式的 URL 访问您的频道列表：

  http://[您的服务器IP或域名]:[STREAMSHIELD_PORT]/[SECURITY_TOKEN]/all.m3u （all.m3u可省略，和老版本保持统一）
  
## 例如：
  http://100.100.100.100:4994/your_security_token/all.m3u

## ⚠️ **强烈建议用 NGINX HTTPS 反向代理一下播放地址和aktv的地址，要注意防止被白嫖。反向代理设置要注意关闭HTTP2，aktv程序不支持HTTP2**


## 设置定时更新 mytvsuper_tivimate.m3u 文件： 为自动化运行，每日早晚执行更新。或遵循 https://pixman.io/topics/17 手动调整。
```bash
(crontab -l 2>/dev/null | grep -v "docker exec pixman sh -c 'flask mytvsuper_tivimate'"; echo "0 5,17 * * * docker exec pixman sh -c 'flask mytvsuper_tivimate'") | crontab -
  ```  
确保所有需要的端口都已开放（默认为 4994, 5000, 4124）
检查 .env 文件中的配置是否正确
如果遇到任何问题，请在 Issues 页面提出。


# Docker 部署指南
1. **预置** Pixman Docker 镜像：

   [https://pixman.io/topics/17](https://pixman.io/topics/17)

2. **加载** StreamShield Docker 镜像：

```bash
   docker pull ppyycc/streamshield-proxy:latest
```
设置定时更新 mytvsuper_tivimate.m3u 文件：
为自动化运行，每日早晚执行更新。或遵循 https://pixman.io/topics/17 手动调整。
(crontab -l 2&gt;/dev/null | grep -v "docker exec pixman sh -c 'flask mytvsuper_tivimate'"; echo "0 5,17 * * * docker exec pixman sh -c 'flask mytvsuper_tivimate'") | crontab -



## 启动 Docker 容器
```bash
docker run -d -p 4994:4994 --name streamshield-proxy \
    -e CUSTOM_DOMAIN="http://100.100.100.100:5000" \
    -e VPS_HOST="http://200.200.200.200:4994" \
    --restart always \
    ppyycc/streamshield-proxy:latest 
```
## 定制环境变量

| 变量 | 描述 |
| --- | --- |
| CUSTOM_DOMAIN | 已运行pixman docker的URL，不需附带 m3u 后缀，已包含 YSP 和 4GTV 等聚合。示例：http://1.1.1.1:5000 或 https://bb.bb.bb |
| VPS_HOST | 个人的 VPS HOST，支持 HTTP/HTTPS，可以是 IP 地址或定制域名。示例：http://2.2.2.2:4994 或 https://cc.cc.cc |
| DEBUG="true" | 是否要开启DEBUG， 不写这个扩展默认不开启。 |
| SECURITY_TOKEN="testtoken" | 输入自己设置的安全token防止扫到端口被爆破。 |
| EXTRA_M3U_URLS | 写入多个别的VPS订阅地址进行多源聚合，多源优先级按照写入先后排定优先级，本机优先级在最后。 |




## 部署案例


### 仅使用 IP 同一VPS地址部署：
```bash
docker run --name thetv -d -p 41244:4124 dtankdemp/thetvapp-m3u:latest
docker pull ppyycc/streamshield-proxy:latest \
docker run -d -p 8888:4994 --name streamshield-proxy \
-e CUSTOM_DOMAIN="http://100.100.100.100:5000" \
-e VPS_HOST="http://100.100.100.100:8888" \
-e SECURITY_TOKEN="test11" \
--restart always \
ppyycc/streamshield-proxy:latest
```

访问地址：http://100.100.100.100:8888/test11/all.m3u， 并已自动导入 mytvsuper_tivimate.m3u，并且能收看大陆电视台。


### 多VPS CDN部署：
把别的VPS生成的订阅链接放在主服务器的config/remote_m3u/no_proxy/sources.txt下便可。

访问地址：https://iptv.bbbb.com/test33， 没有大陆电视台，
效果如下
![image](https://github.com/user-attachments/assets/a7862d7c-fec5-4d9e-9d7e-a67779ed4e7b)


## 媒体终端配置

在直播配置中，如 影视，只需填入 https://iptv.bbbb.com/testtoken 或 http://200.200.200.200:8888/testtoken 即可实现流媒体播放。


## 社区互动与支持

更多详情、问题解决或加群探讨，敬请加入 Pixman 的 Telegram 群组：https://t.me/livednowgroup


## 项目贡献

欢迎代码提交、提出功能需求与错误汇报！有意向贡献者，请访问 问题页面，参与共建。


## 使用协议

本项目遵循 MIT 开源协议。
```
