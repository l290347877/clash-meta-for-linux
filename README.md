[TOC]

# 项目介绍

此项目是通过使用开源项目[clash](https://github.com/Dreamacro/clash)作为核心程序，再结合脚本实现简单的代理功能。

本项目是基于[clash-meta-for-linux](https://github.com/wnlen/clash-meta-for-linux)进行二次开发，使用clash-meta组件，支持`vless`协议，性能有极大改善

# 使用教程

## 下载项目

下载项目

```bash
$ git clone https://github.com/sailing0505/clash-meta-for-linux.git
```

进入到项目目录，编辑`.env`文件，修改变量`CLASH_URL`的值。

```bash
$ cd clash-meta-for-linux
$ vim .env
```

> **注意：** `.env` 文件中的变量 `CLASH_SECRET` 为自定义 Clash Secret，值为空时，脚本将自动生成随机字符串。

<br>

## 启动程序

直接运行脚本文件`start.sh`

- 进入项目目录

```bash
$ cd clash-meta-for-linux
```

- 运行启动脚本

```bash
$ sudo bash start.sh

正在检测订阅地址...
Clash订阅地址可访问！                                      [  OK  ]

正在下载Clash配置文件...
配置文件config.yaml下载成功！                              [  OK  ]

正在启动Clash服务...
服务启动成功！                                             [  OK  ]

Clash Dashboard 访问地址：http://<ip>:9090/ui
Secret：xxxxxxxxxxxxx

请执行以下命令加载环境变量: source /etc/profile.d/clash.sh

请执行以下命令开启系统代理: proxy_on

若要临时关闭系统代理，请执行: proxy_off

```

```bash
$ source /etc/profile.d/clash.sh
$ proxy_on
```

- 检查服务端口

```bash
$ netstat -tln | grep -E '9090|789.'
tcp        0      0 127.0.0.1:9090          0.0.0.0:*               LISTEN     
tcp6       0      0 :::7890                 :::*                    LISTEN     
tcp6       0      0 :::7891                 :::*                    LISTEN     
tcp6       0      0 :::7892                 :::*                    LISTEN
```

- 检查环境变量

```bash
$ env | grep -E 'http_proxy|https_proxy'
http_proxy=http://127.0.0.1:7890
https_proxy=http://127.0.0.1:7890
```

以上步鄹如果正常，说明服务clash程序启动成功，现在就可以体验高速下载github资源了。

- 异常排查

如果 http://<ip>:9090/ui 无法访问，请检查`conf/config.yaml`文件是否正确，或者查看日志文件`logs/clash.log`。

<br>

## 重启程序

如果需要对Clash配置进行修改，请修改 `conf/config.yaml` 文件。然后运行 `restart.sh` 脚本进行重启。

> **注意：**
> 重启脚本 `restart.sh` 不会更新订阅信息。

<br>

## 停止程序

- 进入项目目录

```bash
$ cd clash-meta-for-linux
```

- 关闭服务

```bash
$ sudo bash shutdown.sh

服务关闭成功，请执行以下命令关闭系统代理：proxy_off

```

```bash
$ proxy_off
```

然后检查程序端口、进程以及环境变量`http_proxy|https_proxy`，若都没则说明服务正常关闭。


<br>

## 配置TUN模式（可选示例）
- 手动修改config.yaml文件

```bash
$ vim conf/config.yaml
```

- 添加TUN模式配置

```text
dns:
  enable: true
  ipv6: false
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
  nameserver:
    - 223.5.5.5        # 阿里DNS
    - 119.29.29.29     # 腾讯DNS
    - https://doh.pub/dns-query         # 腾讯 DoH
    - https://dns.alidns.com/dns-query  # 阿里 DoH
  fallback:
    - https://dns.cloudflare.com/dns-query
    - tls://1.1.1.1:853
    - tls://8.8.4.4:853
  fallback-filter:
    geoip: true
    ipcidr:
      - 240.0.0.0/4
      - 0.0.0.0/32

      
tun:
  enable: true
  stack: gvisor
  dns-hijack:
  - any:53
  auto-route: true
  auto-detect-interface: true

```

- 重启服务

```bash
$ sudo bash restart.sh
```


## Clash Dashboard

- 访问 Clash Dashboard

通过浏览器访问 `start.sh` 执行成功后输出的地址，例如：http://192.168.0.1:9090/ui

- 登录管理界面

在`API Base URL`一栏中输入：http://\<ip\>:9090 ，在`Secret(optional)`一栏中输入启动成功后输出的Secret。

点击Add并选择刚刚输入的管理界面地址，之后便可在浏览器上进行一些配置。

- 更多教程

此 Clash Dashboard 使用的是[yacd](https://github.com/haishanh/yacd)项目，详细使用方法请移步到yacd上查询。


<br>

# 常见问题

1. 部分Linux系统默认的 shell `/bin/sh` 被更改为 `dash`，运行脚本会出现报错（报错内容一般会有 `-en [ OK ]`）。建议使用 `bash xxx.sh` 运行脚本。

2. 部分用户在UI界面找不到代理节点，基本上是因为厂商提供的clash配置文件是经过base64编码的，且配置文件格式不符合clash配置标准。

   目前此项目已集成自动识别和转换clash配置文件的功能。如果依然无法使用，则需要通过自建或者第三方平台（不推荐，有泄露风险）对订阅地址转换。
   
3. 程序日志中出现`error: unsupported rule type RULE-SET`报错，解决方法查看官方[WIKI](https://github.com/Dreamacro/clash/wiki/FAQ#error-unsupported-rule-type-rule-set)
