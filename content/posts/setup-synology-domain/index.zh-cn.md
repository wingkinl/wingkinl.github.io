---
title: "在内网和外网使用域名无端口访问群晖及内网其他设备/服务"
date: 2023-04-26T15:44:05+08:00
tags: ["群晖"]
categories: ['life']
lightgallery: true
featuredImagePreview: "nas_portal.png"
draft: true
---

## 设置自定义域名

### 群晖入口的域名

到 **控制面板** 找到 登录门户，在 **域** 下方的 **自定义域** 里输入域名，比如 nas.example.com。

如果有 HTTPS 的证书，那就：

* 把 **启用 HSTS 会强制浏览器使用安全连接** 勾选上
* 把 **网页服务** 下的 **自动将 DSM 桌面的 HTTP 连接重定向到 HTTPS** 勾选上

{{< image src="nas_portal.png" caption="最终效果" width="743">}}

### 群晖的应用程序入口

在 **控制面板**/**登录门户**/**应用程序**，选择要修改的服务，点编辑，在 **域** 下面的 **自定义域** 输入想要的域名。

然后回到路由器的域名劫持设置页面添加一个记录。

{{< image src="synology_app_portal.png" caption="群晖应用自定义域名" width="1052">}}

### 群晖上搭建的网站

在添加虚拟主机的时候，**门户类型** 选择 **基于名称**，在 **主机名** 输入想要的域名，比如 web.example.com。

然后同样回到路由器的域名劫持设置页面添加一个记录。

{{< image src="web_name_based.png" caption="虚拟主机使用域名" width="900">}}

## 内网访问

为了在内网访问，有两种方式：

### 修改 hosts 文件，指向群晖的内网 IP 地址。

在 Windows 上可以使用微软的 **[PowerToys](https://learn.microsoft.com/en-us/windows/powertoys/#hosts-file-editor)** 里面的 **Hosts File Editor** 来编辑。

{{< image src="hosts_edit.png" caption="修改 hosts" width="662">}}

### 修改路由器的域名劫持，指向群晖的内网 IP 地址。

到 **OpenWrt** 的 **网络** 的 **DHCP/DNS** 下，找到 **自定义挟持域名**，添加域名。

还可以顺便在上方的静态地址分配设置 nas 的固定 IP 地址。

{{< image src="dns_hijack.png" caption="自定义挟持域名" width="927">}}

### 奇特的方法

还有一种奇特的做法：到域名解析服务商那里添加一条 A 记录，然后指向群晖的内网 IP 地址 :joy:

## 外网访问

如果有公网 IPv4 地址，那就最好，设置 DDNS 就可以了。可惜现在不一定能拿到公网 IP 了。

如果没有公网 IP 地址，我总结得出两种方法：

* 配置 IPv6 的网络环境，获取公网 IPv6 地址。缺点是在外网访问时使用的设备和所在的网络环境也必须是 IPv6 的。目前手机开流量的时候默认已经支持 IPv6 了。[配置 IPv6 的方法在这里]({{< ref "/posts/setup-ipv6-in-openwrt" >}} "配置 IPv6 的方法")
* 内网穿透。包括 [zerotier](https://www.zerotier.com/)、[tailscale](https://tailscale.com/)、[frp](https://github.com/fatedier/frp) 等等方法。
