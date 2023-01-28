---
layout:     post
title:      The Precision Loss of Double-to-Float Conversion
subtitle:   Incorrect display
date:       2023-01-22
author:     Yves
header-img: "img/post-bg.jpg"
catalog: true
tags:
    - C++ 
    - Qt
    - Issue
---


## Problem
The graph below illustrates the difference in the display scene beteewn our application and AutoCAD. We can see the point-value issue here.
![image](/img/20230122/3.1.png)

## Debugging
This curve are composed of B-splines, so I remove most of them and only keep small portion of the curve to simplify debugging.
The accompanying figure compares the simplified curve in AutoCAD and our application, it clearly illustrates the problematic problem.
![image](/img/20230122/3.2.png)

I have no idea unitl examining the X/Y value of one of the points, the value is large.
![image](/img/20230122/3.3.png)

And another problem is the gap between any two points is extremely small.
![image](/img/20230122/3.4.png)

As before rendering, we have the double-to-float function, combining the large-value and small-gap problem we talk about above. It may raise the precision issue, resulting in rendering two line in scene, not a curve.
![image](/img/20230122/3.5.png)

In this case, from the first to the eighth point, the value remains the same at 884828.875 by the conversion loss. However, from the ninth to last point, the value remains at 884828.9375.
![image](/img/20230122/3.6.png)
![image](/img/20230122/3.7.png)

## Solution
A simple solution for this issue has yet to be found. Using double precision in rendering is not currently feasible. To resolve this, the X/Y values can be kept within 10000 by translating. 
The graph below illustrates the translated curve, which now accurately display the data.
![image](/img/20230122/3.8.png)