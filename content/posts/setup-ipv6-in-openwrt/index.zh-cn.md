---
title: "在 OpenWrt 中设置获取公网 IPv6"
date: 2023-04-25T15:44:17+08:00
tags: ['OpenWrt', 'IPv6', 'IPv4']
categories: ['life']
draft: true
---

## 目的

中国电信不给分配公网 IPv4 地址了，为了让在外网的时候访问家里的群晖，我想要试试看 IPv6 的方式效果怎样。

## 网络环境

我把电信光猫改成了桥接，使用刷了 OpenWrt 的小米路由器来拨号。

## 配置

先上最终配置成功后的接口总览：

{{< image src="result.png" caption="最终结果" width="630" height="758">}}

### 配置 LAN 接口

找到 **网络/接口/LAN**，点修改

#### 基本设置

进入 **基本设置**，**IPv6 分配长度** 确定为 **64**

{{< image src="lan_basic.png" caption="基本设置" width="938" height="1103">}}

#### 高级设置

进入**高级设置**，进行这些改动：

* 取消勾选 **使用内置的 IPv6 管理**
* 勾选 **强制链路**

