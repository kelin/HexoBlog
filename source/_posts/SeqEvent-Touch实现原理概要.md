---
title: SeqEvent_Touch实现原理概要
date: 2016-11-26 17:39:52
categories: 虚幻3
tags:
---

SeqEvent_Touch是kismet的事件节点，Actor被触摸的时候触发。

# Actor的SupportdEvents和GeneratedEvents数组


如图，Actor下有两个数组：SupportdEvents和GeneratedEvents。
![](http://upload-images.jianshu.io/upload_images/3713845-98757182b3d66e44.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1. SupportdEvents是此Actor支持的事件，一般在defaultproperties被初始化。如果需要支持SeqEvent_Touch，则必须加上。
2. GeneratedEvents则保存了发生关联的事件。
在关卡启动，kismet序列被初始化的时候，会将事件添加到对应的Originator的GeneratedEvents数组。具体逻辑的主要函数如下：
![](http://upload-images.jianshu.io/upload_images/3713845-376271e0dad18c8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3713845-b5831331d80cf876.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3713845-c9eba499a888e50d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](http://upload-images.jianshu.io/upload_images/3713845-842d4346e766fc75.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
USequeneceEvent是SeqEvent_Touch的父类。而事件就是在RegisterEvent()函数与对应的Originator关联起来的。

# Actor被Touch后的逻辑
SeqEvent_Touch有如下几个cpp函数，从函数名字面不难理解其实现的主要功能：
![](http://upload-images.jianshu.io/upload_images/3713845-d148ad3aefe09440.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当Actor被触摸后，会执行到下图的函数，可以看到，这个函数找到Actor所关联的所有SeqEvent_Touch事件，执行它的CheckTouchActivate()函数。然后执行eventTouch()，也就是unrealscript中的Touch函数。
![](http://upload-images.jianshu.io/upload_images/3713845-1f4441ba412e8dae.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)