---
title: "Open repo in WSL with Git Extensions"
date: 2023-05-17T14:04:24+08:00
draft: false
categories: ['tech']
tags: ['Git', 'WSL', 'OpenWrt']
featuredImage: "WSL-Git-Extensions_cover.png"
toc: false
---

I cloned a repo in WSL in order to build an OpenWrt firmware, but I couldn't open it with Git Extensions:

![Error](error.png)

Even after clicking on the **Trust this repository** button, it still fails to open.

I fixed it by manually running this command:

```powershell
git config --global --add safe.directory '%(prefix)///wsl.localhost/Ubuntu/home/kenny/immortalwrt-mt798x'
```

Reference:

https://github.com/gitextensions/gitextensions/issues/9954
