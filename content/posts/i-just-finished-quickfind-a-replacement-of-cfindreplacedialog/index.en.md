---
title: 'I just finished QuickFind, a replacement of CFindReplaceDialog!'
date: Wed, 29 Mar 2017 16:10:00 +0000
draft: false
toc:
  enable: false
tags: ['C++','MFC']
categories: ["work"]
featuredImage: "quickfind.gif"
lightgallery: true
---

I really like the design and look of Quick Find box in Visual Studio, it give you the impression of instant result, automatically jumps to the first match, and it does not block you active editor UI.  
  

{{< image src="vsquickfind.png" caption="Quick Find in Visual Studio" width="332" height="133">}}

  
Recently I needed to add the missing regular expression option in Find and Replace dialog, so I figured why not try to implement my own version of Quick Find, see how it goes, and even better: I can add some features that are not available in VS. For example, I sometimes want to resize the Quick Find box not just horizontally but also vertically. One of my colleagues even suggests it would be great if the Quick Find box allows him to move it around, not just sticking/docking at the upper right corner.  
  
After 2~3 weeks (after work and weekends), I finally made this:  
  

{{< image src="quickfind.gif" caption="My implementation of Quick Find" width="444" height="144">}}

  
[Visit the Git repo here](https://github.com/wingkinl/QuickFind)