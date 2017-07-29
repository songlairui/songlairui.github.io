---
title: 初尝试WebSocket
date: 2017-07-07 21:16:41
tags:
---
> 尝试Socket，使用之中有很多问题。  

首先，在图解HTTP一书中，知道http头部有个connection，能够用来将http协议升级到websocet。  
然后，在node环境下，使用http模块启动web server，让能够直接处理http头部。    

现在，要使用websocket了，那么，是不是将要接触将http 升级到 websocket的细节了？  
不，倾向于使用下载量超级多的npm包。这些包之中，应该做了升级协议的细节。  

<!--more-->

## ‘君子性非异也，善假于物也’  

筛选到的包有 `ws`,`faye-websocket`。 这两个包的下载量最多。  
浏览了下包的首页介绍，其中前者可以像浏览器中一样，使用现成的WebSocket API。  
后者，在web-dev-server中的hmr相关代码中发现有依赖到。浏览器介绍，能看出是在http模块上进行处理。  

## `ws` 的 express 例子  

README中的express例子简短的展示了与express结合使用的基本法。  
未能演示如何使用拿到的ws。  
在github repo中的example中，[服务器状态](https://github.com/websockets/ws/tree/master/examples/serverstats) 这个例子，展示了前后端使用websokcet通讯的使用。

## 使用 `ws` 从前端发送指令，到后端执行  

代码: [https://github.com/songlairui/amarscfpack](https://github.com/songlairui/amarscfpack)  中的 server.js (node后端) 与 static(前端)  

前端先使用 (`connectWS` 方法)[https://github.com/songlairui/amarscfpack/blob/master/static/websocket.part.js]建立websokcet连接, 并做了断线重连的操作  

然后使用 `sendCmd` 方法，按照格式发送JSON.stringify处理完成的指令。 

```javascript
...
  ws.send(JSON.stringify(Object.assign(command, { taskid })))
...
``` 

后端接受后，使用JSON.parse解析的到指令内容，进行判断并执行相应指令  

```javascript 
...
  ws.on('message', function incomint(message) {
    console.info(`received: ${message} `)
    dealWithMsg(ws, message)
  })
...
```  


### 其他细节 

发送数据时，添加任务id，异步事件处理完成后，通知前端时加上此标示id，执行前端回掉函数。  
详细: {% post_link js-promise-transform %}

前端通过`ws`发送String简便易行，在后端拿到buffer更为方便。后端往前端传输buffer时，想带上额外的标示信息怎么办？  
使用了一种将buffer stringify，然后放在 {} 中再stringify以传输。降低了效率，完成了传输。  
如果直接传输buffer，没有标示信息，需要在前端添加机制，约定好buffer的位置。这种机制未尝试，但尝试了前后端传输 buffer 的细节处理  
详细: {% post_link transfer-buffer-with-ws %}

## TO LEARN  

需要了解http请求头的字段，在websocket中解析的使用方式。
 

