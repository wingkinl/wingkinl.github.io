---
title: 'Just spent one Saturday and implemented this Goto Related Symbols thing'
date: Sun, 01 Jul 2018 14:58:11 +0000
draft: false
toc:
  enable: false
tags: ['C++']
categories: ["work"]
featuredImage: "gotorelated.gif"
images: ["gotorelated.gif"]
---

I've always wanted a feature like this in our built-in code editor, it can make understanding code structure a simpler job. The recorded gif above is pretty straightforward and tells everything without me saying too much to explain why it is so useful.

Anyway, I decided to implement this context menu for quickly jumping to related classes/functions at type of current caret position. I thought this should be an easy job, one Saturday afternoon would be enough. But I was wrong. I spent one whole Saturday and even worked late till 2 P.M, this has happened a lot in the past, especially when I have the enthusiasm to implement something I think useful and fun to code. The Derived Classes part is a bit tricky, since we don't have such knowledge in the compiled symbol object (i.e. there is no array of pointers to the derived classes), I have to loop through all the known class types and check if they are derived from the given base class, and if so, add them to the derived classes array.

Another tricky part of this is that I need to show the derived class as a "hierarchy" popup menu, meaning I need to somehow show the depth of each class type with indentation such that developer can better make sense of the hierarchy relationship. The array of derived classes are stored without any order, I have to build a tree base on that array, then do depth-first traverse the tree and reconstruct another array sort that array based on their relative depth (i.e. compare two class type using some method like IsDerivedFrom(), or compare using class name) for building the popup menu.