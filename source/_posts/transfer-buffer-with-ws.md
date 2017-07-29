---
title: 通过ws包前后端相互传输ArrayBuffer/Buffer
date: 2017-07-29 13:16:54
tags:
---

> 在node中，使用 child_process.spawn 或 fs.readFile 直接拿到的是 Buffer。 
使用ws直接传输buffer想来会更高效率。  

<!--more-->

**占坑** 

# 后端 node

## Buffer的读取  

node读取到buffer后，设定缺省编码 'utf8'. 
一般的api中可以手动设定为 'binary' 以二进制方式读取buffer。  

读取到的 buffer 有大小。一般的，api中能够设置buffer的单元大小。比如 highWaterMark  

## Buffer的基本格式  

```javascript
new Buffer([0xdf,0xea]) 
```

拿到buffer之后，其在内存中的存储就像这样  


## Buffer的 Stringify 

使用JSON.stringify() 将buffer转换为字符串，再使用JSON.parse() 解析，会得到格式如 `{ type: 'buffer', buffer: [...]}` 的对象。  
根据Buffer的基本格式，可以重塑该Buffer 

## Buffer的拼接 

## Buffer的编码  

# 前端 chrome  

Chrome中没有Buffer对象，有ArrayBuffer 

## 操作ArrayBuffer  

new Uint8Array
