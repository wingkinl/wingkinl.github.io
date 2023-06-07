---
title: "使用密钥连接 SSH"
date: 2023-04-30T10:22:15+08:00
categories: ['tech']
tags: ['SSH']
featuredImage: "ssh_key_cover.png"
draft: false
---

## 生成密钥对

不同的操作系统其实过程基本一样，微软也提供了详细步骤：

https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_keymanagement

用管理员权限运行 **PowerShell**。

运行 **ssh-keygen** 来生成 RSA 密钥对。

```powershell
PS C:\Users\kenny> ssh-keygen
```

按提示需要输入保存文件的路径。我输入了 **id_rsa_test**

```
Enter file in which to save the key (C:\Users\kenny/.ssh/id_rsa): C:\Users\kenny\.ssh\id_rsa_test
```

接着要输入密码对这个密钥加密保护

```
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in C:\Users\kenny/.ssh/id_rsa_test.
Your public key has been saved in C:\Users\kenny/.ssh/id_rsa_test.pub.
The key fingerprint is:
SHA256:OIzc1yE7joL2Bzy8!gS0j8eGK7bYaH1FmF3sDuMeSj8 kenny@LOCAL-HOSTNAME

The key's randomart image is:
+--[RSA 3072]--+
|        .        |
|         o       |
|    . + + .      |
|   o B * = .     |
|   o= B S .      |
|   .=B O o       |
|  + =+% o        |
| *oo.O.E         |
|+.o+=o. .        |
+----[SHA256]-----+
```

把这个密钥添加到 ssh-agent 以便以后不需要在连接 SSH 的时候输入密码。

在使用 **ssh-add** 加入ssh-agent 之前，Windows 用户还有一些额外的步骤要做：

```powershell
# ssh-agent 服务默认是禁用的，需要配置成自动开启
# 确保在管理员权限下执行这个代码
Get-Service ssh-agent | Set-Service -StartupType Automatic

# 启用服务
Start-Service ssh-agent

# 查看服务状态
Get-Service ssh-agent

# 把这个密钥添加到 ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_rsa_test
```

## 部署公钥

### 部署到群晖

1. 登录到群晖管理页面
2. 打开 **File Station** > **home**.
3. 新建名为 **.ssh** 的文件夹
4. 上传公钥 **id_rsa.pub** 到 **.ssh** 文件夹

{{<image src="upload_key_to_dsm.png" width="100%">}}

更多说明可查看：

https://kb.synology.com/en-uk/DSM/tutorial/How_to_log_in_to_DSM_with_key_pairs_as_admin_or_root_permission_via_SSH_on_computers

### 部署到 OpenWrt

1. 打开 **系统 → 管理权 → SSH 密钥**
2. 复制公钥文件的内容。 公钥内容以 `ssh-rsa ...` 开头，并以 `... some-name@some-host.lan` 结尾
3. 把公钥文件的内容粘贴到下方的输入框
4. 为了测试是否有效，可在命令行执行 `ssh root@路由器地址`，成功的话应该无需密码直接连上了

更多说明可查看：

https://openwrt.org/docs/guide-user/security/dropbear.public-key.auth#from_the_luci_web_interface

## SSH 连接方法

SSH 的默认端口是 22。如果修改过这个端口号，则需要在连接的时候指定：

* ssh ssh://user@address:端口
* ssh -p 端口 user@address

## 删除私钥

在删除之前要启动 `ssh-agent`：

```
eval `ssh-agent -s` 
```

删除全部私钥：

```
ssh-add -D
```

删除指定的私钥：

```
ssh-add -d ~/.ssh/sshkeynamewithout.pub
```

列出目前的私钥：

```
ssh-add -l
```

更多说明可查看：

https://stackoverflow.com/a/73572251

