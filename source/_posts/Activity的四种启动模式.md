---
title: Activity的四种启动模式
date: 2017-01-13 22:05:28
categories: Android  
tags: [Android, Activity, 启动模式]

---
# Activity的四种启动模式
## 1.Standard
standard是默认Activity的启动模式。standrad模式的Activity的无论是否已经存在在返回栈中，系统都会创建一个新的Activity。
## 2.SingleTop
在singleTop模式下，已经存在在返回栈栈定中的Activity可以直接使用而无需重新创建。
## 3.SingleTask
使用singleTak可以很好的解决重复创建栈顶活动的问题。避免使用singTop模式是Activity不存在与返回栈栈顶时重复创建多个Activity实例的问题。当Activity启动模式指定为singleTask时，每次启动该Activity时系统都会在返回栈中检查是否存在该活动的实例，如果有则会直接使用，并把这个活动之上的活动统统出栈，如果没有怎会新建一个实例。
## 4.SingleInstance
实验该模式的activity会使用单独的返回栈（在我的三星s7 edge 7.0测试系统上出现了使用该模式的的activty和未使用该模式的的Activity使用同一个返回栈的问题)。
