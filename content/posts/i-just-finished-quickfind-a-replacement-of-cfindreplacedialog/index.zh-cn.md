---
title: '我今天实现了 CFindReplaceDialog 的替代版 QuickFind！'
date: Wed, 29 Mar 2017 16:10:00 +0000
draft: false
toc:
  enable: false
tags: ['C++','MFC']
categories: ["work"]
featuredImage: "quickfind.gif"
images: ["quickfind.gif"]
lightgallery: true
---

我真的很喜欢 Visual Studio 里的查找工具，搜索迅速，实时跳转到选中的结果，而且界面小巧不阻挡编辑内容。
  
{{< image src="vsquickfind.png" caption="Visual Studio 的 Quick Find" width="332" height="133">}}

  
我最近需要实现 Find and Replace 对话框里缺失的正则表达式功能，所以我就想何不干脆自己实现一个 Quick Find，看看能做成怎样，而且我还能自己添加一些连 VS 都没有但我们一直想要的功能。比如，有时候我想要调整 Quick Find 工具的大小：不仅仅是横向调整宽度，还想在垂直方向直接调整显示内容。有个同事甚至建议说如果可以随意移动它就更好了（VS 的只能固定在编辑器的右上角）。
  
我花了2~3周的下班时间（包括周末），成品如下：
  

{{< image src="quickfind.gif" caption="我实现的 Quick Find" width="444" height="144">}}

  
[我把源代码放在这里了](https://github.com/wingkinl/QuickFind)