# StreamShield Proxy: pixman HLS MPD流媒体播放代理工具

## 简介

StreamShield Proxy 的目标是解决因 IP 限制而无法直接播放 pixman.io 上的 4gtv.m3u 等流媒体文件的问题。Cloudflare 免费版的流媒体代理功能缺失，并且存在封号风险，因此本项目利用个人 多个VPS（例如甲骨文的 ARM 服务器）作为CDN多源聚合代理，流畅转发所需流量，显著简化了流媒体播放的配置流程。它尤其适用于家中没有卵路由器或难以安装复杂配置环境的用户。

## 核心功能

- **智能代理:** 支持 4gtv.m3u、Beesport.m3u、mytvsuper.m3u 和 TheTV。即使在 IP 受限的情况下，依然能够流畅播放4GTV和 MytvSuper。为了流畅观看 MytvSuper，您需要在 pixman Docker 环境中自行配置相应的key。
- **内容聚合:** 集成央视屏、中国移动 iTV、蜀小果、江苏移动魔百盒和 TPTV（直连）。您可以通过开关选择是否要导入这些直连电视，默认不开启。聚合了各种热门内容，如央视节目、中国移动 iTV、蜀小果、江苏移动魔百盒等，扩展您的内容访问范围。
- **多源支持:** 支持多VPS部署后聚合成CDN网络，可在终端播放软件电视台内进行多源切换。
- **加固安全:** 新增安全 token，有效防止服务被未授权扫描利用。
- **简化安装:** 提供直观便捷的流媒体配置流程，极大降低了部署难度。
- **兼容性优化:** 支持 arm64 和 amd64 架构。
- **自定义 M3U 导入:** 您可以先在 pixman docker 中导入自定义 M3U。由于目前还没有相关需求，因此我还没有测试过这个功能，如果您遇到任何问题，请在社区反馈。
- 可与pixman部署在同一个vps上，IP写同样便可。

## 发展现状

最新版 StreamShield Proxy 已集成绝大多数 Pixman 渠道，并支持自定义 M3U 导入，除了 YouTube。

由于 Mytvsuper 使用 mpd 加密技术连接，每次 IPTV 换台的时间大约是 4gtv 的四倍，加重了换台等待感。

支持Thetv的HLS加密方式。

在 Android 环境下，您需要使用 [https://github.com/FongMi/Release/tree/fongmi/apk/release](https://github.com/FongMi/Release/tree/fongmi/apk/release) 支持 mpd 加密解码播放。

## Docker 部署指南

1. **预置** Pixman Docker 镜像：

   [https://pixman.io/topics/17](https://pixman.io/topics/17)

2. **加载** StreamShield Docker 镜像：

   ```bash
   docker pull ppyycc/streamshield-proxy:latest

设置定时更新 mytvsuper_tivimate.m3u 文件：
为自动化运行，每日早晚执行更新。或遵循 https://pixman.io/topics/17 手动调整。
(crontab -l 2&gt;/dev/null | grep -v "docker exec pixman sh -c 'flask mytvsuper_tivimate'"; echo "0 5,17 * * * docker exec pixman sh -c 'flask mytvsuper_tivimate'") | crontab -



## 启动 Docker 容器

docker run -d -p 4994:4994 --name streamshield-proxy \
    -e CUSTOM_DOMAIN="http://100.100.100.100:5000" \
    -e VPS_HOST="http://200.200.200.200:4994" \
    -e SECURITY_TOKEN="test11" \
    -e INCLUDE_MYTVSUPER="free" \
    -e CHINAM3U="true" \
    --restart always \
    ppyycc/streamshield-proxy:latest 

## 定制环境变量

| 变量 | 描述 |
| --- | --- |
| CUSTOM_DOMAIN | 已运行pixman docker的URL，不需附带 m3u 后缀，已包含 YSP 和 4GTV 等聚合。示例：http://1.1.1.1:5000 或 https://bb.bb.bb |
| VPS_HOST | 个人的 VPS HOST，支持 HTTP/HTTPS，可以是 IP 地址或定制域名。示例：http://2.2.2.2:4994 或 https://cc.cc.cc |
| INCLUDE_MYTVSUPER="true"/"free" | 是否启用 mytvsuper_tivimate.m3u 渠道加载，缺省不加载。有自己的key就用ture，没有key就用free看12个免费电视台 |
| DEBUG="true" | 是否要开启DEBUG， 不写这个扩展默认不开启。 |
| SECURITY_TOKEN="testtoken" | 输入自己设置的安全token防止扫到端口被爆破。 |
| CHINAM3U="true" | 是否要开启大陆电视台， 不写这个扩展默认不开启。 |
| CUSTOM_M3U=“test.m3u”| 是否要开倒入自定义M3U，名字和pixman docker内一致。 |
| CUSTOM_M3U_PROXY="true" | 是否要用本程序代理自定义M3U流量，不写这个扩展默认默认不开启自定义M3U代理。 |
| CUSTOM_M3U_PROXY_HOST | 写入自定义M3U需要代理的host，方便程序识别并代理。 |
| EXTRA_M3U_URLS | 写入多个别的VPS订阅地址进行多源聚合，多源优先级按照写入先后排定优先级，本机优先级在最后。 |


## 部署案例


仅使用 IP 同一VPS地址部署pixman和代理：


docker pull ppyycc/streamshield-proxy:latest \
docker run -d -p 8888:4994 --name streamshield-proxy \
-e CUSTOM_DOMAIN="http://100.100.100.100:5000" \
-e VPS_HOST="http://100.100.100.100:8888" \
-e SECURITY_TOKEN="test11" \
-e INCLUDE_MYTVSUPER="true" \
-e CHINAM3U="true"
--restart always \
ppyycc/streamshield-proxy:latest


访问地址：http://100.100.100.100:8888/test11， 并已自动导入 mytvsuper_tivimate.m3u，并且能收看大陆电视台。


仅使用 IP 地址部署并只看free mytvsuper：


docker pull ppyycc/streamshield-proxy:latest \
docker run -d -p 8888:4994 --name streamshield-proxy \
-e CUSTOM_DOMAIN="http://100.100.100.100:5000" \
-e VPS_HOST="http://200.200.200.200:8888" \
-e SECURITY_TOKEN="test22" \
-e INCLUDE_MYTVSUPER="free" \
-e CHINAM3U="true"
--restart always \
ppyycc/streamshield-proxy:latest


访问地址：http://200.200.200.200:8888/test22， 并已自动导入 mytvfree.m3u，并且能收看大陆电视台。


搭配域名和 HTTPS 部署：

docker pull ppyycc/streamshield-proxy:latest \
docker run -d -p 444:4994 --name streamshield-proxy \
-e CUSTOM_DOMAIN="https://pixman.aaaa.com" \
-e VPS_HOST="https://iptv.bbbb.com" \
-e SECURITY_TOKEN="test33" \
--restart always \
ppyycc/streamshield-proxy:latest
访问地址：https://iptv.bbbb.com/test33， 默认不包含 mytvsuper 频道清单，没有大陆电视台。

多VPS CDN部署：

docker pull ppyycc/streamshield-proxy:latest \
docker run -d -p 444:4994 --name streamshield-proxy \
-e CUSTOM_DOMAIN="https://pixman.aaaa.com" \
-e VPS_HOST="https://iptv.bbbb.com" \
-e SECURITY_TOKEN="test33" \
-e EXTRA_M3U_URLS='https://iptv.cccc.com/test123,https://iptv.dddd.com/test123'
--restart always \
ppyycc/streamshield-proxy:latest
访问地址：https://iptv.bbbb.com/test33， 默认不包含 mytvsuper 频道清单，没有大陆电视台，聚合别的两个VPS作为多源源头。别的两个vps部署上可以不用加'EXTRA_M3U_URLS'参数启动，只是作为单源播放点。优先级1.cccc 2.dddd 3.bbbb
效果如下
![image](https://github.com/user-attachments/assets/a7862d7c-fec5-4d9e-9d7e-a67779ed4e7b)




## 媒体终端配置

在直播配置中，如 影视，只需填入 https://iptv.bbbb.com/testtoken 或 http://200.200.200.200:8888/testtoken 即可实现流媒体播放。


## 社区互动与支持

更多详情、问题解决或加群探讨，敬请加入 Pixman 的 Telegram 群组：https://t.me/livednowgroup。


## 项目贡献

欢迎代码提交、提出功能需求与错误汇报！有意向贡献者，请访问 问题页面，参与共建。


## 使用协议

本项目遵循 MIT 开源协议。
```
