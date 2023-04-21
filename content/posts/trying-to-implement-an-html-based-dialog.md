---
title: 'Trying to implement an HTML-based dialog'
date: Sat, 18 Jun 2016 15:47:00 +0000
draft: false
toc:
  enable: false
tags: ['HTML', 'COM']
categories: ["work"]
---

To implement a new feature of the software, I need to make an HTML-based dialog. Initially, I thought this should be easy. Later, when I consider the subtle details, such as passing data between C++ and javascript and the localization issues, it appears to be not that easy.  
  
Base on the learning from the search result of Google, it is not a simple job when it comes to passing an object/array from javascript to C++. A common solution I've seen so far is to figure out a way to pack/unpack the object/array as an XML/JSON string arguments. XML seems to be a quite suitable solution to me, because we can easily deal with it in both C++ and Origin C (OC). The only problem left is to dispatch the function calls to from javascript to C++, then to OC.  
  
Since OC already support handling VARIANT object, casting different types of data has also been implemented, so directly passing the arguments as VARIANT\* to OC would be the simplest way of doing this job. However, Using existing fundamental data types such as int/double/string/etc is still preferable. So I had been busy struggling with this subtle issues for the past whole week. Hopefully the job will be done in the next week.