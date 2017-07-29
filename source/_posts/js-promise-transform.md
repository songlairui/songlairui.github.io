---
title: 想了一下，Promise可以这样写
date: 2017-07-29 13:16:34
tags:
---

>  Promise 变体。 
代码示例脱胎于 [execSequence](https://github.com/songlairui/amarscfpack/blob/master/static/index.js#L75)  方法 与 [TaskSequence.prototype.executeOne](https://github.com/songlairui/amarscfpack/blob/master/static/TaskSequence.component.js#L20) 方法。  
与常见的promise相比，变化的地方在于，then(Function) 的参数中的 resolve 被存到了数组中。  
<!--more-->
## 占坑 