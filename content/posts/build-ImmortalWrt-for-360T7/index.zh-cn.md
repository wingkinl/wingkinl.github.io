---
title: "为 360T7 编译 ImmortalWrt 固件"
date: 2023-05-17T10:11:27+08:00
categories: ['tech']
tags: ['ImmortalWrt', 'OpenWrt', 'IPv6', 'WSL']
featuredImagePreview: "images/ImmortalWrt.png"
images: ["images/ImmortalWrt.png"]
draft: false
---

## 起因 

原本使用小米的 AC2100 搭配 N1 作为旁路由，一切正常，后来折腾 IPv6，发现这种环境出现各种问题。比如 YouTube 能打开，但是无法播放视频。 因为我在别的地方使用小米的 AX3600 作为主路由使用一切正常，于是决定在闲鱼买了 360T7，卖家还改了512M 的内存。 可惜卖家刷好的 OpenWrt 固件似乎不太稳定，重启后 WiFi 会断开必须重新连接。 为了找到合适的固件，我在恩山论坛上找到了 uparrow 的这篇帖子：

{{< showcase title="[2023.04.27] 360T7 lede 23.4.1 |IPV6|猫咪|多播|酸奶|Upnp|adguardhome|全千兆" summary="由于官方lede目前没有对360T7进行开源，故移植hanwckf的mt7981的dts和配置到lede的源码，构建出lede版本。" image="https://www.right.com.cn/forum/uc_server/data/avatar/000/66/04/06_avatar_middle.jpg" link="https://www.right.com.cn/forum/thread-8282287-1-1.html" column=1 >}}

不过作者并没有在固件里加入 **passwall**，所以我希望自己编译把 **passwall** 包括进去，同时去掉不需要的软件包。作者的代码在 [GitHub](https://github.com/uparrows/immortalwrt-mt798x) 上，基于 **hanwckf** 的 [immortalwrt-mt798x](https://cmi.hanwckf.top/p/immortalwrt-mt798x/) 项目修改的。

## 编译环境搭建

我是在 Windows 11 下安装 WSL 2，在 ubuntu 22.04 LTS 下编译的。

### 安装 WSL

在 PowerShell 下运行这个命令安装：

```powershell
wsl --install
```

安装完要重启系统。

### 配置 WSL 系统资源

先关闭 WSL：

```powershell
wsl --shutdown
```

打开配置文件：

```powershell
notepad "$env:USERPROFILE\.wslconfig"
```

按照个人需要添加下面的内容：

```powershell
[wsl2]
memory=10GB  
processors=4
```

### 禁止 WSL 的 ubuntu 使用 Windows 的环境变量

根据网上的[说法](https://blog.csdn.net/weixin_43698781/article/details/124792708)：

> 使用WSL中的工具时，恰好windows中有同名的，虽然在wsl中设置了PATH，但还是有些会找到windows中的，从而报错。

下面引用修改的方法：

在wsl 的 ubuntu中编辑/etc/wsl.conf，输入：

```ini
[interop]
enabled = false
appendWindowsPath = false
```

在 WSL ubuntu 中使用 echo $PATH 查看 PATH，如果没有 Windows 自己的东西就正常了。

### 配置 Ubuntu 22.04 的国内源

#### 备份Ubuntu官方的软件源，执行以下命令

```bash
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
```

#### 更新使用清华大学ubuntu 22.04镜像

```bash
sudo bash -c "cat << EOF > /etc/apt/sources.list && apt update 
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
EOF"
```

### 配置依赖库

运行下面的代码：

```bash
sudo apt update -y
sudo apt full-upgrade -y
sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
  bzip2 ccache clang clangd cmake cpio curl device-tree-compiler ecj fastjar flex gawk gettext gcc-multilib \
  g++-multilib git gperf haveged help2man intltool lib32gcc-s1 libc6-dev-i386 libelf-dev libglib2.0-dev \
  libgmp3-dev libltdl-dev libmpc-dev libmpfr-dev libncurses5-dev libncursesw5 libncursesw5-dev libreadline-dev \
  libssl-dev libtool lld lldb lrzsz mkisofs msmtp nano ninja-build p7zip p7zip-full patch pkgconf python2.7 \
  python3 python3-pip python3-ply python-docutils qemu-utils re2c rsync scons squashfs-tools subversion swig \
  texinfo uglifyjs upx-ucl unzip vim wget xmlto xxd zlib1g-dev
```

## 开始编译

### 建立本地仓库并更新 feeds：

```bash
git clone https://github.com/uparrows/immortalwrt-mt798x
cd immortalwrt-mt798x
./scripts/feeds update -a
./scripts/feeds install -a
# 对于 mt7981，推荐使用 mt7981-ax3000.config
cp -f defconfig/mt7981-ax3000.config .config
make menuconfig
```

360T7 是 mt7981 的，所以使用 mt7981-ax3000.config

### 配置编译参数

我选了 ZeroTier 和 passwall 这两个包，其它基本上没有改动。

### 编译

运行 `make V=s` 开始编译固件