---
title: 读KOA文档 - 马后炮心思
date: 2017-12-24 09:32:09
tags: koa
---

> 今天早晨，将剩下的一半KOA官方网站读完了。  

### 花花心思  
昨天晚上读到request/reponse列表就睡觉了，早上读完发现，剩下的都是这一部分。因为属性太多，每个都介绍一下，占用的篇幅长。   
这让我觉得对自己很气，因为之前一直因为懒，不愿意读完官方文档，尝试了很多koa教程，也没有完成，没想到官方文档这么简易。  

所以，我期望用数据呈现每一部分占的篇幅。当一个章节占用篇幅过大时，一可能是真多，二可能是api全列表描述。  
怎么判断列表呢？  //TODO  

#### 篇幅占比计算
> 假设文档长度即作者想给看到的长度。（如果正文中有折叠，也是作者想给到的折叠）

1. 获取目录  

首先，使用chromeDevTools 添加：hover 让菜单常显示。  
获取目录DOM元素  
```javascript
var els = document.querySelectorAll('#menu ul li a')
```

2. 获取标题

```javascript
var titleEls = [].map.call(els,el=>document.querySelector(el.getAttribute('href')))
titleEls
/* 
 (6) [h1#introduction, 
      h1#application, 
      h1#context, 
      h1#request, 
      h1#response, 
      h1#links]
*/
```

3. 获取篇幅比例 
3.1 方式1 - 根据点获取篇幅比例 
> 如果不存在可用div恰好包裹每个章节，则可以通过title位置计算篇幅。最后一个章节的结束位置，要视情况而定。
[示意图]

```javascript
// 获取所有标题的最小公共父元素
let target = titleEls[titleEls.length - 1]
while(!target.querySelector('#'+titleEls[0].id)){
  // console.group('while')
  // console.log('target',target)
  if(!target.parentNode){
    // console.groupEnd()
    break;
  }
  target = target.parentNode
  // console.log('target2',target)
  // console.groupEnd()
}
if(!el) throw new Exception('无有效公共父元素')
// 计算父元素总高度（不精准, 不效率）

let cachedStyleHeight = target.style.height
let cachedStyleOverFlow = target.style.overflow
target.style.height = 'auto'
target.style.overflow = ''
let height = target.offsetHeight
target.style.height = cachedStyleHeight
target.style.overflow = cachedStyleOverFlow

```

3.2 根据可用的包裹content元素获取

> 如果存在恰好包裹每个章节的父元素

``` javascript
// 获取最高非共用父元素
function findEl(el,otherSelector){
  let target = el
  while(target.parentNode && !target.parentNode.querySelector(otherSelector)){
    if(!target.parentNode){
      target = null
      break
    }
    target = target.parentNode
  }
  // 如果父元素包含另外一个标题的selector，则返回当前target
  return target
}

var chapterEls = titleEls.map((el,idx,els)=>{
  // 获取另外一个标题所在DOM，如果最后一个元素，取上一个
  let siblingHash = '#' + els[ idx+1+1>els.length ? idx-1 : idx+1].id
  return findEl(el,siblingHash)
})

```

4. 计算篇幅占比
4.1 处理3.1数据
```javascript
// 计算父元素总高度（简单）
let contentHeight = target.scrollHeight
// 拼接标准输出数据
let data = titleEls.map(el=>el.offsetTop)
data.push(contentHeight)

let result = data.reduce((r,c,i,a)=>{
 if(i<a.length - 1){ // 非最后一个元素
   r[i]={}
   r[i].start = c
 }
 if(i>0){ // 如果有上一个元素，就填充end，并计算length
   r[i-1].end = c 
   r[i-1].length = r[i-1].end - r[i-1].start
   r[i-1].percent = Math.round((r[i-1].end - r[i-1].start)/contentHeight *10000) /100
 }
 return r
},[])

```
4.2 处理3.2数据

```javascript
let contentHeight = (el=>el.offsetTop + el.offsetHeight).call(null,chapterEls[chapterEls.length - 1]) // 
let result = chapterEls.map(el=>({
  start:el.offsetTop,
  length:el.offsetHeight,
  end:el.offsetTop + el.offsetHeight,
  percent:Math.round(el.offsetHeight / contentHeight * 10000) / 100
}))
```

5. 打印结果 

```javascript
console.table(result)
```

### MORE IDEA  

1. 标示阅读位置  
在目录元素中，添加篇幅占比。  
为当前阅读位置，做标示。  
检测阅读动作。  
  当快速划过，或快速浏览时，判定为未阅读。
  当连续阅读，没有停顿时，判定为非常规阅读。（阅读速度必然有变化）

2. 抓取目录链接之后，过程容错
- a标签容错
- titleEl 容错
- content length 聚类 - 着重提示较长content