---
title: 'Snap windows'
date: 2024-02-28T23:18:00+08:00
draft: false
tags: ['UI', 'C++', 'Animation']
categories: ["work"]
featuredImage: "cover.png"
---

Some users in the company forum complained that it was not easy to arrange document windows so that they can nicely tile with each other. Although there is already a built-in tile feature provided by the MFC framework, they want a way to resize adjacent windows all together. I was assigned to implement this in Origin and I immediately realize two things:

1) This is somewhat due to the limitation of Origin still using the MDI framework. All document windows are contained inside the same MDI client area, users then have to struggle to arrange those windows within that limited display area. Even though nowadays most people (especially in a working environment) are using multiple displays, Origin still could not take advantage of that and allow showing document windows simultaneously on different screens (I proposed to support dragging windows out of MDI client area, and I implemented it later).
2) Windows has similar feature that we can simply follow how it works and implement it.

Microsoft Windows added the Snap Assist feature in Windows 10 for easily organizing windows:

[Snap your windows - Microsoft Support](https://support.microsoft.com/en-us/windows/snap-your-windows-885a9b1e-a983-a3b1-16cd-c531795e6241)

The implementation is basically done by handling WM_ENTERSIZEMOVE and WM_EXITSIZEMOVE messages, combined with the necessary WM_MOVING and WM_SIZING. I also tried different ways of rendering the windows: alpha blended layered window (through UpdateLayeredWindow) and Direct Composition ([DirectComposition - Win32 apps | Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/directcomp/directcomposition-portal)). In the end I chose Direct Composition for obvious reason: performance and smooth visual effects.

And here is some demonstrations:

![SnapWindowsAnimation](SnapWindowsAnimation.gif)



![Snap Windows To Border](https://blog.originlab.com/wp-content/uploads/2022/03/SnapWindowsToBorder.gif)

![Snap to empty space](https://blog.originlab.com/wp-content/uploads/2022/03/SnapToGray.gif)

![Resize windows in group](https://blog.originlab.com/wp-content/uploads/2022/03/SnapWindowsGreen.gif)





[New Features in Origin 2022b: Arrange and Snap Windows – Origin Blog (originlab.com)](https://blog.originlab.com/new-features-in-origin-2022b-arrange-and-snap-windows)
