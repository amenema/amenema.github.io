---
layout:     post
title:      "node事件循环(Event Loop)"
subtitle:   " \"Event Loop 原理\""
date:       2017-05-18 12:00:00
author:     "Mzx"
header-img: "img/post-bg-2015.jpg"
catalog: true
keyword: JS,nodeJS,node,event loop,event,loop,事件循环队列,时间循环,io,IO,异步
description: 本文描述了nodeJS的event loop是如何运行的，其原理是什么。
tags:
    - NODE
---


# **Node**事件循环(Event Loop) 

> **nodeJS**是如何实现异步**I/O**的呢？什么是事件驱动？揭开**nodeJS**的神秘面纱，还原‘事件’真相！  

## **NodeJS**架构  

什么是**nodeJS**？  
> **nodeJS**是一个基于**Chrome V8**引擎的运行时。  

![](http://7xwfcm.com2.z0.glb.qiniucdn.com/img/u1O2O.png) 

* **V8** : 高性能**javascript**引擎。
* **libuv** : 提供异步功能的基于**C**的库。其运行时包含一个**event loop**,一个线程池和各种**I/O**组件。
* **zlib**,**c-ares**等**C/C++**组件：这些组件用于对系统底层的功能访问。
* **Bindings** : 用于**JS**与原生的**C/C++**进行数据交互。即上面提到的**C/C++**组件是通过**Bindings**才能让**JS**访问到。
* **Addons** : 如果你自己实现了一个**C/C++**库，那么**JS**该如何操作这个库呢？这里就需要你自己实现**JS**与**C/C++**库的**Bindings**代码，你写的**Bindings**代码即**C/C++ Addons**
* **Node.js API** : js代码 

## **如何运行**  
上面分析了**nodeJS**都有哪些组件，那这些组件又是如何工作的呢？ 

![](http://7xwfcm.com2.z0.glb.qiniucdn.com/img/WX20170521-214044.png)  

### step 1: 
  
**V8**引擎将**JS**编译为机器码，然后执行，如果遇到**I/O**操作（网络请求，文件读写等），则将该操作所需的参数和回调方法打包成一个请求对象发给**libuv**   

### step 2:

**libuv**接收到该对象后将其放入线程池，如果线程池中有空闲线程可以执行该操作，则分发一个线程进行处理。处理完毕后，将得到的结果绑定到该对象上，然后将其放入修改该对象的状态为已执行完毕，然后归还线程   

### step 3:

**Event Loop** 在**nodeJS**启动时，**libuv**便会启动一个类似**while (true) {}**的无限循环，其会不停地轮询**时间**线程池中已经执行完毕的对象，当获取到执行完毕的对象时，便将请求对象放入到**I/O**观察者队列中，然后获取对象中绑定的结果和回调函数，执行回调函数并将绑定的结果作为参数传入。至此，便是一个完整的异步**I/O**调用流程。  

## 事件循环（**Event Loop**）  
事件循环就是个循环，其不断检查是否有事件待处理，如果有，就处理该事件。如果没有则进入下一个循环。在每个循环中判断是否有事件需要处理是通过观察者判断的。


## 总结  

* 事件循环，观察者，请求对象，**I/O**线程池四者是一个完整异步**I/O**模型的必备要素。
* **V8**是单线程，事件循环是单线程，但是**libuv**在处理**I/O**时是多线程的。
