---
title: Android Broadcast原理解析
tags: [Android,Broadcast]
grammar_cjkRuby: true
categories: [Android]
date: 2017-08-09
---

> 说明：该文所有内容均来自网络，这里只是简单汇总一下。

### 原理概述

**==Android中的广播使用了设计模式中的观察者模式：基于消息的发布/订阅事件模型，不过，同时利用消息队列+Handler机制实现广播的线性化（保证广播不会丢失）==**

模型中有3个角色：
消息订阅者（广播接收者）
消息发布者（广播发布者）
消息中心（AMS，即Activity Manager Service）

![enter description here][1]


广播接收者 通过 Binder机制在 AMS 注册
广播发送者 通过 Binder 机制向 AMS 发送广播
AMS 根据 广播发送者 要求，在已注册列表中，寻找合适的广播接收者

AMS将广播发送到合适的广播接收者相应的消息循环队列中；
广播接收者通过 消息循环（Handler） 拿到此广播，并回调 onReceive()
特别注意：广播发送者和广播接收者的执行是异步的，发出去的广播不会关心有无接收者接收，也不确定接收者到底是何时才能接收到；

### 注册流程
![enter description here][2]

![enter description here][3]

### 广播流程
![enter description here][4]

![enter description here][5]

![enter description here][6]


### 关于Binder机制
Binder机制用来实现进程间通信（IPC），是Android核心机制之后 ，类似的机制如Handler机制实现线程切换和消息循环。

![enter description here][7]

![enter description here][8]

### 参考资源

http://www.jianshu.com/p/abb173858faf
http://www.cnblogs.com/asi24/p/4314280.html
http://blog.csdn.net/luoshengyang/article/details/6737352
http://blog.csdn.net/luoshengyang/article/details/6744448

  [1]: ./images/1502265387429.jpg
  [2]: ./images/1502268846515.jpg
  [3]: ./images/1502268813211.jpg
  [4]: ./images/1502268837583.jpg
  [5]: ./images/1502268677030.jpg
  [6]: ./images/1502268776173.jpg
  [7]: ./images/1502267152492.jpg
  [8]: ./images/1502267906247.jpg