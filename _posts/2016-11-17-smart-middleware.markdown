---
layout:     post
title:      "koa中间件之smart-middleware"
subtitle:   " \"自动匹配路由中间件\""
date:       2016-11-17 12:00:00
author:     "Mzx"
header-img: "img/post-bg-2015.jpg"
catalog: true
keyword: Koa,node,smart-middleware,middleware,router,
description: 本文介绍了koa的中间件smart-middleware的使用方法。
tags:
    - Node Koa
---


#  smart-middleware
> smart-middleware 是 koa下的一个用于自动加载路由，并自动根据配置好的规则批准路由对应中间件的中间件。  
 

[![Travis](https://img.shields.io/badge/npm-0.1.1-brightgreen.svg?style=flat-square)](https://www.npmjs.com/package/smart-middleware)[![Build Status](https://travis-ci.org/amenema/smart-middleware.svg?branch=master)](https://travis-ci.org/amenema/smart-middleware)[![Coverage Status](https://coveralls.io/repos/github/amenema/smart-middleware/badge.svg?branch=master)](https://coveralls.io/github/amenema/smart-middleware?branch=master)[![npm](https://img.shields.io/npm/l/express.svg?style=flat-square)](https://github.com/amenema/smart-middleware/https://github.com/amenema/smart-middleware/blob/master/LICENSE)
 
## 项目地址
[npm](https://www.npmjs.com/package/smart-middleware)  
[github](https://github.com/amenema/smart-middleware)

## 安装  

```
npm install smart-middleware
```
## 使用  

```
/*step 1 app.js*/
var sm = require('smart-middleware');
var router = require('koa-router')();
var app = require('koa-router')();

var middlewares = [
                   {url: '/list',
                     fn: [function *(next){
                       "use strict";
                       this.body += '_m_1';
                       yield next;
                     }, function *(next){
                       "use strict";
                       this.body += '_m_2';
                       yield next;
                     }
                     ]},
                   {url: '\\^(?!/open)', fn: [function *(next){
                     "use strict";
                     this.body = '_m_3';
                     yield next;
                   }, function *(next){
                     "use strict";
                     this.body += '_m_4';
                     yield next;
                   }]}
                   ,{url: '\\(/open)', fn: [function *(next){
                     this.body = '_m_1';
                     yield next;
                   }]}
                 ];
sm.autoLoading({router: router/*required*/, middleware:middlewares, path:__dirname +'/routers'/*required absolute path*/});
app.use(router.routes());


/*step 2 /routers/user.js*/
module.exports = function(router){
  router.get('/list', function *(next){
    this.body += '/list';
  });
  router.get('/open/user', function *(next){
    this.body += '/open/user';
  });
};
```  

### 注：
1. **smart-middleware**只有一个方法**autoLoading(router, middleware, path)** 。其中**router**是**koa-router**必须传入，**path**是项目的路由文件夹的**绝对路径**,也是必须传入。**middleware** 是管理项目中间件的数组。默认为**[]**。
2. 参数**middleware**的格式： 
 
	```
	 template: {url: 'url', fn: [m1,m2]}
	```  
3. 路由匹配规则：  
	
	```
	url: (url.indexOf('\') === 0)? 'this is regexp' : 'this is common String
	```  
	
	* 字符串完全匹配
	* 正则匹配：正则写法，在**url**中加入**‘\\\’**表示该**url**使用正则匹配
4. **middleware**匹配规则
	* 加载中间件时根据url从前往后匹配，形成一个匹配成功的数组，如： 
	 
	```
	[{url:'/list', fn:[m_1, m_2]}, {url:'\\^(?!/open)', fn: [m_3, m_4]}]
	```  
	,则当访问路由**/list**时, 匹配上的中间件的执行顺序为: **m\_3, m\_4, m\_1, m\_2**

## 测试  

```
npm test
```

## 反馈   

* 评论区留言
* [github-issue](https://github.com/amenema/smart-middleware/issues)