---
title: "在群晖安装 Portainer 并设置域名访问"
date: 2023-04-28T15:49:38+08:00
tags: ['群晖', 'Docker', 'Portainer']
categories: ["tech"]
featuredImage: "cover.png"
draft: false
---

这篇仅供我日后参考使用。

## 创建容器

SSH 连接到群晖，并执行下面的代码：

```bash
sudo -i

mkdir -p /volume1/docker/portainer

docker create --name=portainer --privileged --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v /volume1/docker/portainer:/data portainer/portainer-ce
```



## 启用网络入口并设置域名

在 Docker 里找到 portainer 容器后选择编辑，勾选 **通过 Web Station 启用网页门户**

添加端口，指定 **9443** 为 HTTPS

{{< image src="web_portal.png" caption="设置网页门户" width="100%">}}

然后输入域名即可

{{< image src="custom_web_portal_domain.png" caption="定义域名" width="100%">}}
