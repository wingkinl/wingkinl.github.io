---
title: "在内网和外网使用域名无端口访问群晖及内网其他设备/服务"
date: 2023-04-26T15:44:05+08:00
tags: ["群晖", "IPv6", "OpenWrt", "frp", "ZeroTier", "Cloudflare"]
categories: ['life']
lightgallery: true
featuredImagePreview: "nas_portal.png"
images: ["nas_portal.png"]
draft: false
---

## 目的

域名肯定是比 IP 地址好记的，如果加上端口号那就更显得累赘。为了在内网和外网都使用域名访问内网的设备，并且尽量不需要端口号，我折腾了一下各种方法。

## 设置自定义域名

### 群晖入口的域名

到 **控制面板** 找到 登录门户，在 **域** 下方的 **自定义域** 里输入域名，比如 nas.example.com。

如果有 HTTPS/SSL 的证书，那就：

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

上面提到的设置域名的方式，由于群晖的这些服务共用 80/443 端口，那域名就是唯一一个用来区分访问的服务的方式了。

为了在内网访问，有两种方式：

### 修改 hosts 文件，指向群晖的内网 IP 地址。

在 Windows 上可以使用微软的 **[PowerToys](https://learn.microsoft.com/en-us/windows/powertoys/#hosts-file-editor)** 里面的 **Hosts File Editor** 来编辑。

{{< image src="hosts_edit.png" caption="修改 hosts" width="662">}}

### 修改路由器的域名劫持，指向群晖的内网 IP 地址。

到 **OpenWrt** 的 **网络** 的 **DHCP/DNS** 下，找到 **自定义挟持域名**，添加域名。

还可以顺便在上方的静态地址分配设置 nas 的固定 IP 地址。

{{< image src="dns.png" caption="自定义挟持域名" width="927">}}

### 奇特的方法

还有一种奇特的做法：到域名解析服务商那里添加一条 A 记录，然后指向群晖的内网 IP 地址 :joy:

## 外网访问

### 设置防火墙

首先肯定要先设置防火墙，否则外网是连不通的。

OpenWrt 的防火墙默认是拒绝来自外网 wan 口的数据的：

{{< image src="openwrt_firewall.png" caption="防火墙默认禁止外网" width="1167">}}

#### 添加通信规则

在 OpenWrt 的 **网络**/**防火墙**/**通信规则** 页面下，添加一条 **转发规则**。

源区域选择 wan，目标区域选择 lan。然后根据需要设置地址和端口等等，端口号用空格分开。例如为了让外网可以通过 443 和 8443 端口访问，可以如下设置：

{{< image src="openwrt_firewall_rule.png" caption="防火墙的通信规则" width="786">}}

### 外网访问的各种方式

如果有公网 IPv4 地址，那就最好，设置 DDNS 就可以了。可惜现在不一定能拿到公网 IP 了。

如果没有公网 IP 地址，我总结得出两种方法：

* 内网穿透。包括 [zerotier](https://www.zerotier.com/)、[tailscale](https://tailscale.com/)、[frp](https://github.com/fatedier/frp) 等等方法。

* 配置 IPv6 的网络环境，获取公网 IPv6 地址。缺点是在外网访问时使用的设备和所在的网络环境也必须是 IPv6 的。目前手机开流量的时候默认已经支持 IPv6 了。[配置 IPv6 的方法在这里]({{< ref "/posts/setup-ipv6-in-openwrt" >}} "配置 IPv6 的方法")

#### 通过 frp 访问

frp 的缺点是需要一个在公网上的服务器做转发，网上可以找到免费的 frp 服务器，有些服务器支持 80/443 端口转发到内网，那就实现免端口号了。

在 OpenWrt 上安装 frpc 客户端，然后到服务端上添加一个服务器

{{< image src="frp_server.png" caption="frp 服务端设置" width="725">}}

到规则页面，添加一条规则：

代理名称注意输入一个能保证和别人不一样的（因为是共享的）

类型选择 HTTPS

本地 IP 输入群晖的内网 IP 地址

本地端口可以输入 443

自定义域名输入群晖服务的域名，比如 web.example.com

{{< image src="frp_rule.png" caption="frp 代理规则设置" width="664">}}

网络速度取决于公网 frp 服务器的速度，使用上不太理想。要么使用免费但速度不稳定的服务器，要么就要自己购买 VPS。

#### 通过 Zerotier 访问

Zerotier 和 tailscale 的原理相似，需要按照服务器端和客户端。我注册了账号并且创建了一个网络，然后在两个不同地点的 OpenWrt 上连接到这个网络，进行适当的配置，就可以像在内网一样访问类似 192.168.2.1 这样的设备地址了。

##### 速度测试

Zerotier 的速度挺好的，对于 30m 的上传带宽来说基本上是满速了。

{{< image src="zerotier_download_speed.png" caption="zerotier 的下载测试" width="407">}}

在外网运行 zerotier 的客户端连接上后，也一样可以通过修改 hosts 文件的方法实现通过域名访问了。

#### 通过 lucky 的 STUN 内网穿透访问

在 OpenWrt 安装 lucky 后，打开 stun 穿透，会得到一个 IP:端口 的公网可用地址。

速度非常快，可以达到满速，可惜端口号不能自己决定。

项目地址：[https://github.com/gdy666/lucky](https://github.com/gdy666/lucky)

#### 通过 Cloudflare Zero Trust 的隧道访问

cloudflared 的项目地址：[https://github.com/cloudflare/cloudflared](https://github.com/cloudflare/cloudflared)

速度也还不错，从一开始的500KB/s慢慢爬升到1700KB/s。而且可以创建很多域名，并且可以绑定端口号（当然也可以不设置端口号），访问的时候是不需要端口号的。

{{< image src="cloudflare_tunnel_speed.png" caption="cloudflared 的下载测试" width="499">}}

#### 通过 IPv6 访问

[配置好 IPv6 的环境后]({{< ref "/posts/setup-ipv6-in-openwrt" >}} "配置 IPv6 的方法")，可以到域名解析服务商添加一条 **AAAA** 记录。比如 Cloudflare。**不过国内 80 和 443 端口都被禁了**。

由于 IPv6 不一定所以网络环境下都可用（比如公司网络就可能不支持），考虑到有时需要在纯 IPv4 的环境下使用，就只能想别的方法了。

Cloudflare 针对这种情况允许通过它的代理进行 IPv4 到 IPv6 的访问，打开小云朵就可以了，缺点就是会慢。另外，Cloudflare 的[官方文档](https://developers.cloudflare.com/fundamentals/get-started/reference/network-ports/)说只支持代理下面这些端口：

HTTP 端口：

* 80
* 8080
* 8880
* 2052
* 2082
* 2086
* 2095

HTTPS 端口：

* 443
* 2053
* 2083
* 2087
* 2096
* 8443

为了考虑不同场合的使用需要，我添加了两条记录，一条直连，一条通过代理。

{{< image src="cf_dns_ipv6.png" caption="Cloudflare 的 DNS 记录" width="1168">}}

由于默认的端口 80/443 被禁，不能简单地实现无端口访问。要访问必须修改端口。有以下两种方法：

* 在群晖修改对应的端口。对于虚拟主机，打开编辑页面，修改对应端口即可。

* 或者到 OpenWrt 上做端口映射，从外部的一个端口比如 8443 转到群晖的 443.

然后就可以在外网（无论是否支持 IPv6）通过 https://web.example.com:8443 来访问群晖上的虚拟服务器了。如果确定外网环境支持 IPv6，就访问 https://webv6.example.com:8443

因为 IPv6 地址会变，这种方法还需要设置 DDNS。

##### 速度测试

经过实测，在支持 IPv6 的外网访问可以达到宽带的带宽上限，通过 Cloudflare DNS 代理的速度则比较不稳定，但总体来说比 Cloudflare 隧道还是要快，有时候还可以满速。Cloudflare 隧道的方式我试过不同时间访问，速度都不是很理想。

家里的宽带是电信 300m下载，30m上传。直连 IPv6 的时候基本上可以达到 4MB/s 的速度，也就是满速了。

## 总结

目前来看，在没有公网 IP 地址的情况下，能实现在外网访问且无端口号的方式有 frp、zerotier、cloudflared。如果要求安全，zerotier可能比较适合，但在外网使用也需要客户端。如果需要让别人无需客户端也能访问内网的服务，比较理想的可能还是 cloudflared 了。
