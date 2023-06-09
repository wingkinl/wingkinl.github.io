---
title: 'The window-resizing-flickering nightmare, part 1'
date: Thu, 14 Jun 2018 10:30:35 +0000
draft: false
tags: ['C++']
categories: ["work"]
featuredImage: "tabflickerkeyframes_cover.gif"
lightgallery: true
---

## The issue

A user complained that some dialog appears too big, specifically, on his/her high DPI screen (150%, 1920x1080), too high that the buttons on the bottom of the dialog are simply out of the screen, as shown below: ![DialogTooHigh.png](dialogtoohigh2.png "Bottom part of the dialog is outside of the screen")

As you see, the dialog almost takes up the whole screen, users will have to move the dialog around a bit to be able to click on the buttons at the bottom of the dialog. What we are seeing here is a property sheet dialog ([CPropertySheet](https://docs.microsoft.com/en-us/cpp/mfc/reference/cpropertysheet-class), with a tab control in it), which consists of a few property pages. When the property sheet is initialized, it calculates the largest space enough to show all the property pages at any given time. This is quite convenient for us in most cases, but the problem is quite obvious too: some property pages do not require so much space, especially in a scenario like this high DPI (but low resolution) environment. 

## Solution
An ideal solution to this may be:

1) to rearrange the property pages so that roughly all of them are of the same height, and

2) make sure none of the pages is too high.

Since we've been using this property sheet dialog for quite a long time, more and more pages were added to the dialog, it certainly will take a lot of time to rearrange all of them. Instead of rearranging all of the pages, we decided to resize the dialog to fit (to the smallest possible size of) the active page whenever users switch pages. Good design or not, at least it allows users to activate another smaller page when they encounter a similar scenario. So the idea is quite simple, handles the [PSN_SETACTIVE](https://msdn.microsoft.com/en-us/library/windows/desktop/bb773169(v=vs.85).aspx) notification, then calculate the needed size of the active page, then resize the windows (tab control, property page, property sheet) accordingly.

It goes like this:

{{< highlight cpp >}}
void CDemoPropSheet::OnActivatePage(CPropertyPage* pPage)
{
    CSize szPage = CalcPageCompactSize(pPage);
    CRect rectPage(CPoint(0, 0), szPage);
 
    CRect rectTab = rectPage;
    GetTabControl()->AdjustRect(TRUE, &rectTab);
 
    CRect rectDialog = rectTab;
    // add margins to rectDialog here
 
    // fix the rectangles correspond to the client are
 
    GetTabControl()->SetWindowPos(...)
    GetActivePage()->SetWindowPos(...)
    SetWindowPos(...)
}
{{< /highlight >}}


### Some trouble

What could possibly go wrong? As it turns out that doing so causes quite a bit flickering on the tab control. To demonstrate the issue, I created a [demo project](https://github.com/wingkinl/TabCtrlFlickerDemo) based on the source code of the [NewControls](https://github.com/Microsoft/VCSamples/tree/master/VC2010Samples/MFC/Visual%20C%2B%2B%202008%20Feature%20Pack/NewControls) project from Microsoft's VCSamples. Although the flickering is not always significant and also not consistent, I managed to create this screenshot as gif to show it here. ![TabFlickering.gif](tabflickering.gif)


What exactly has happened here? Let's break it down to the 3 keyframes when we click on the tab. ![TabFlickerKeyframes.gif](tabflickerkeyframes.gif)

The above gif shows before, during, and after the first tab is activated by mouse click. I debugged through this and finally realized that the flickering happens when GetTabControl()->SetWindowPos() is called, or more specifically when the size of the tab control is changed by SetWindowPos calls. In fact, if you set a breakpoint on the **GetTabControl()->SetWindowPos()** call, and step over that line, you can see the tab control disappear (and reappear after that)!

But what else can I use to resize the tab control? **BeginDeferWindowPos/DeferWindowPos** stuff does not help here. [WS\_EX\_COMPOSITED mentioned by some people](https://stackoverflow.com/questions/4188306/flicker-free-tab-control-with-ws-ex-composited) also is not helping, not to mentioned **WM_SETREDRAW** or **LockWindowUpdate** (which is not at all for this purpose).

### Some hope
However, I notice later that if we resize the tab control when the parent of it (property sheet dialog) is being resized, the annoying flickering is gone! ![TabSmoothResizing.gif](tabsmoothresizing.gif)

### Disappointment
This gives me the idea that instead of resizing the tab control directly, I could try resizing the property sheet and later resize the tab control inside the **WM_SIZE** handler. Soon after that, I was disappointed to find out the flickering still exist!

### Another way
What if we don't do the resizing when handling **PSN_SETACTIVE**, but instead when we explicitly invoke via a command such as a button? So I put the code to the "Fit" button's click handler, and it works without much flickering! It seems that it's too early to do such kind of resizing in the **PSN_SETACTIVE** notification, maybe a delayed call can help to mitigate the flickering. It did. You can try the [demo](https://github.com/wingkinl/TabCtrlFlickerDemo) here, remember to turn on the delay sizing as shown below. ![DelaySizing.png](delaysizing.png)