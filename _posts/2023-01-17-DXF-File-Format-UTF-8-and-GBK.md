---
layout:     post
title:      DXF File Format:UTF-8 and GBK
subtitle:   Encoding format
date:       2023-01-17
author:     Yves
header-img: "img/post-bg.jpg"
catalog: true
tags:
    - C++ 
    - Qt
    - Issue
---

## Problem
When importing a DXF file, chinese characters may appear as messy codes and the distance between them is incorrect. (Note: This latter issue will not be addressed in this context.)
![image](/img/20230117/4.2.png) 

The correct display in AutoCAD.
![image](/img/20230117/4.1.png) 

## Analysis
The DXF text is initially converted from std::string to QString use [QString::fromUtf8](https://doc.qt.io/qt-6/qstring.html#fromUtf8-3) function without considering file encoding format. This causes messy code issues when the DXF file use the GBK encoding format.


## Solution
Check if the string format is UTF-8 or GBK(forget screenshot the code!!). If is GBK, use [QString::fromLocal8Bit](https://doc.qt.io/qt-6/qstring.html#fromLocal8Bit).
![image](/img/20230117/4.3.png) 


## Fixed
![image](/img/20230117/4.4.png) 
