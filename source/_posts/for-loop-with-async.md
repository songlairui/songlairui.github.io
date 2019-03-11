---
title: Stream-with-for-loop-with-async
date: 2017-10-16 13:42:45
tags:
 - async
 - for
 - javascript
---

> 我在写一个server端， 【github repo】  
  当我向socket读取数据时，希望降低频率。  
  

### 一个什么样的Socket？  
在android shell中运行minicap「github repo」 ，得到的屏幕刷新数据流。  
android 设备屏幕一旦有刷新，minicap就会将屏幕显示的完整画面，通过socket传输。  

与常见socket的区别：
  和fs.readStream对比，这个socket的读取不是一次性读完的，而是持续产生并输出的。

这个socket的用法，通过websocket Server转发这里的socket，实现通过浏览器显示手机屏幕。

## 关键代码   

```javascript
function tryRead(){
  for (var chunk; (chunk = stream.read()); ) {
    // 省略
  }
}
```

我想控制这里for循环的fps。  
这里`stream.read()`将会一直读取socket，如果没有返回则会等待返回，一旦完成，则继续下一个。  
于是，`for`的大括号里的代码，无法影响到for循环的进程。

## 改动方式一，不理想

```javascript
function tryRead(){
  
  let lastStamp = null
  for (var chunk; (chunk = stream.read()); ) {
    let currentStamp = + new Date()
    if(lastStamp){
      if(currentStamp - lastStamp < 100) return
    } else {
      lastStamp = currentStamp
    }

    // 省略
  }
}
```


## 可能成功的改动