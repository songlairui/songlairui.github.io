---
title: 细致显示歌词
date: 2017-06-27 09:27:18
tags: 
---
> 仿照网易云音乐播放界面的一个页面： [https://songlairui.github.io/NeteaseMusic/static](https://songlairui.github.io/NeteaseMusic/static)  
跟随音乐显示歌词，用了两种方式实现：ontimeupdate事件、循环调用setTimeout。   
**有gif，流量党慎点。**   

<!--more-->

## Step 0 前期操作  

- 页面使用了动态REM，js代码中用了反引号（不支持UC）。
- 图片素材取自网易云音乐手机版页面。  
- 布局使用flex。
- 旋转动画的暂停与播放使用animation-play-state属性控制。
- 歌词存放在json中，使用fetch获取，并split成数组，然后遍历生成DOM并插入页面（使用-了document.createDocumentFragment小小优化）。

**生成的歌词DOM结构**  

```html 
<div class="lyric">
  <ul style="transform: translateY(1px);">
    <li></li>
    <li></li>
    <li data-stamp="00:01.36" class="current">野子 (Live) - 苏运莹</li>
    <li data-stamp="00:02.787">词：苏运莹</li>
    <li data-stamp="00:04.86">曲：苏运莹</li>
    <li data-stamp="00:10.31"></li>
    <li data-stamp="00:14.31">怎么大风越狠</li>
    <li data-stamp="00:17.31">我心越荡</li>
    <li data-stamp="00:20.231">幻如一丝尘土</li>
...
- .lyric 包裹ul，并且设置`overflow:hidden`，可显示区域容纳5行歌词
- 前两个li是手动添加的空li，用于占位
- 每一个li包裹一句歌词
- data-stamp 是歌词中的时间
- ul包裹所有的li 
```


## 需求
基本需求：跟随歌曲播放显示歌词  
更多需求：歌词严格跟随播放进度


## Step 1 完成基本需求

1**. 歌词激活-样式逻辑**  
当指定歌词激活后，为其添加标识样式 '.current',CSS中为此样式设定显著的颜色和阴影。  
计算当前歌词的相对父元素的高度，调节translateY使其在第三行位置（歌词显示区域中间位置）。  
*关键代码:*    
```javascript
// target 是获取的需要激活的歌词
// lrcEl 是包裹每一句歌词`li`的`ul`
target.classList.add('current')
// 令激活的歌词，位于第三行位置
lrcEl.style.transform = `translateY(${- target.offsetTop + 2 * target.offsetHeight}px)`
```
*显示效果:*  
![激活指定歌词](https://of87cyikt.qnssl.com/blog/hexo/activeLrcItem.gif)

2**. 使用进度条常用方法**  
为HTMLMediaElement制作自定义进度条时，会用到 timeupdate 事件。[//TODO:MDN]()

| ontimeupdate| - | 控制台 |
| ------| ------ | ------ |
| ![代码](https://of87cyikt.qnssl.com/blog/hexo/ontimeupdate-1.png) | - | ![效果](https://of87cyikt.qnssl.com/blog/hexo/ontimeupdate.gif) |
| 为audio设置事件| - | 点击播放后，控制台会按照audio的频率打印当前播放时间 | 

3**. 筛选出当前应该激活的歌词**

在updateLrc方法中，筛选出来当前激活的歌词。  

updateLrc 代码1:  
```javascript
function updateLrc() {
  //获取当前audio播放的时间
  let currentStamp = audio.currentTime
  let lastLrc = lrc.filter(v => v[0]) // 清空掉没有时间参数的歌词
    .filter(v => {
      let tmp = v[0].split(':')
        // 修复计算错误，添加括号，优先进行隐式类型转换
      let stamp = 60 * (+tmp[0]) + (+tmp[1])
      return stamp > currentStamp
    })[0]
  console.info('当前要激活的歌词:', lastLrc)
    // 如果找到了指定歌词，就激活它
  if (lastLrc) activeLrcItem(lastLrc[0])
}
```
显示效果:  

| 页面| - | 控制台 |
| ------| ------ | ------ |
| ![歌词激活](https://of87cyikt.qnssl.com/blog/hexo/updateLrc-r1-1.gif) | - | ![控制台信息](https://of87cyikt.qnssl.com/blog/hexo/updateLrc-r1.gif) |
| 选取到指定歌词，然后激活| - | 控制台输出，每出发一次timeupdate，都更新一下要激活的歌词。 | 

这儿配合声音听的话，歌词早了一句，代码先修正，再分析。   

updateLrc 代码2:
```javascript
function updateLrc() {
  let currentStamp = audio.currentTime
  let lastLrc = lrc.filter(v => v[0])  
    .reverse()                        // 这儿添加一行反向
    .filter(v => {
      let tmp = v[0].split(':')
      let stamp = 60 * (+tmp[0]) + (+tmp[1])
      return stamp < currentStamp     // 这儿大于号变小于号
    })[0]
  console.info('当前要激活的歌词:', lastLrc)
  if (lastLrc) activeLrcItem(lastLrc[0])
}
```

- 当前激活的歌词，应该是已经激活过的最后一句歌词。
- 添加了一行 reverse，更改了下filter中的对比条件。  
- 逻辑从原来都取得第一个比当前时间大的歌词，变成了取最后一个比当前时间晚的歌词。  
- 代码中，`filter`返回的是一个二维数组，末尾加`[0]`即取第一个值。  
 - 末尾如果添加`[0][0]`，则在下边判`null`的时候，会报错。  
- `activeLrcItem` 设计这样的传参用法，能同时兼容另一种歌词策略。  

updateLrc 代码3 【添加新方法，增加可读性】:
```javascript
function updateLrc() {
  let currentStamp = audio.currentTime
  let lastLrc = lrc.filter(v => v[0])
    .reverse()
    .filter(v => lrcTime2Second(v[0]) < currentStamp)[0]
  if (lastLrc) activeLrcItem(lastLrc[0])
}
/**
 * 【辅助函数】根据歌词中的时间戳，返回秒数
 * @param {lrc 歌词中的时间戳} lrcTime 
 */
function lrcTime2Second(lrcTime) {
  let tmp = lrcTime.split(':')
    // 修复计算错误，添加括号，优先进行隐式类型转换
  return 60 * (+tmp[0]) + (+tmp[1])
}
```

在`filter`写了三行的代码逻辑被简化了。  
为其创建额外方法之后，将`filter`变成了简写状态，私以为代码可读性提高了。

## Step 2 更多需求: 每个歌词都生效  

> 使用ontimeupdate，依赖audio元素自身事件的机制。如果ontimeupdate的频率太慢，两个事件间隔之间，更新了多个歌词的情况下，歌词的激活就会出现skip。  可以使用30倍速播放音乐，查看ontimeupdate方法等表现。  

**30倍速播放** [DEMO](https://songlairui.github.io/NeteaseMusic/static)    

| Step 1 方法| - | 新方法 |
| ------| ------ | ------ |
| ![timeupdate x30](https://of87cyikt.qnssl.com/blog/hexo/ontimeupdatex30.gif) | - | ![no-skip](https://of87cyikt.qnssl.com/blog/hexo/settimeoutx30.gif) |
| 会出现跳词| - | 每句歌词都被激活 |   

另一个细节是，歌词中空行和下一句歌词的时间间隔很小。使用ontimeupdate方法，激活空行（令其显示在中间位置）的概率很小（是个概率事件）。  
而使用settimeout方法，激活空行，是必然事件。  

**实现细节:**  

准备全局变量(未进行组件化，粗鲁的使用全局变量了):  
```javascript
let timer = null // settimeout需要使用的timer
```

开始播放时，执行一个方法操作歌词:  

```javascript  
function playLrc() {
  if (timer) return
  let currentStamp = audio.currentTime
  // 获取当前激活的歌词，和下一个要激活的歌词， 此处正向获取。因为下一个歌词还没有播放，等待setTimeout延迟激活，
  let nextLrc = lrc
    .filter(v => v[0]) // 清空掉没有时间参数的歌词
    .filter(v => lrcTime2Second(v[0]) > currentStamp)[0]
  if (nextLrc) {
    console.info(`下一歌词:${nextLrc}`)
    timer = setTimeout(function() {
      clearTimeout(timer), timer = null
      activeLrcItem(nextLrc[0])
      playLrc() // 尾调用自身
    }, (lrcTime2Second(nextLrc[0]) - (+currentStamp)) * 1000 / audio.playbackRate)
  }
}
```  

- 取得需要歌词的逻辑，跟上一种实现方式中updateLrc类似。不过，playLrc自身实际操作是在setTimeout超时后，所以，传入的歌词参数应该是未来的一个歌词。恰好使用第一次想到的filter逻辑。  
- 得到nextLrc之后，设置`timer`,并在setTimeout中函数真正执行时，clear掉，并置null。然后在尾部调用自身。  
- 这样的setTimeout是一个套路，没什么可说的。 其中实际执行的是 `activeLrcItem(nextLrc[0])` 这句。  
- setTimeout 的超时时间，根据未来一句歌词，和音乐当前播放的时间求出。所以，每次setTimeout都是新鲜计算的，不会累计时间误差。  
- 超时时间 除以 `audio.playbackRate`, 匹配不同播放速度下切换行为。  
- // 别设置`audio.playbackRate`为‘负’值，就gg了。。
- // 每次遍历读取DOM，需要优化吗？

## 伪END  
好快就说完了，记得做的时候，来回调了特别多遍。  
DEMO地址： [https://songlairui.github.io/NeteaseMusic/static](https://songlairui.github.io/NeteaseMusic/static)  

## 附加  
上述两方法实现过程，还有很枝外细节，写在最后。

### **【缓存变量】激活歌词时，需要遍历去除其他歌词DOM上的激活状态**  

准备全局变量:  

```javascript  
let activedLrcEl = new Set() // cache for 激活的歌词
```  

| 未使用缓存变量 | - | 使用了缓存变量 |
| ------| ------ | ------ |
| ![withoutCacheVar](https://of87cyikt.qnssl.com/blog/hexo/activeItemWithoutCache.png) | - | ![WithCacheVar](https://of87cyikt.qnssl.com/blog/hexo/activeItemWithCacheVar.png) |
| 使用filter方法对DOM遍历读取 | - | 每次激活li，都把其放到Set中，并在取消激活时删除掉 |   

使用缓存变量之后，很起来很骚的长行代码去掉了。  
为什么用Set？因为不用考虑去重了。

| **使用效果:** |
| ------ |
| ![cache效果](https://of87cyikt.qnssl.com/blog/hexo/withCache.png) |
| **settimeout方法下，使用30倍速度播放，查看控制台。此时因为setTimeout的误差，造成超时时间取得过短，playLrc方法执行频次远超实际需要，造成额外尝试操作DOM。缓存变量的使用，会使得其在需要操作DOM时，真正去执行。**|

### **【CSS动画控制】 播放结束歌词处理**  

歌词播放完之后，想让它跳回头部，而不是滚回头部。  
这样处理的想法时，歌曲结束之后，.3s的时间滚回头部，如果看到觉得太黏腻，不利落。
这儿用到void el.clientWidth。算是一种黑魔法？

| 切换 visibility | - | 切换 display |
| ------| ------ | ------ |
| ![viaVisibility](https://of87cyikt.qnssl.com/blog/hexo/onended1.png) | - | ![viaDisplay](https://of87cyikt.qnssl.com/blog/hexo/onended2.png) |
| 无效 | - | 生效 |   

使用display:none会达到想要的效果。猜测跟浏览器绘制过程有关。

### **【CSS】歌词空行的处理**  

默认情况，空的一行会有一个小的height，因为字体高度为0。
让空的一行也显示有内容，就可以获得正常的高度。为每个歌词添加伪元素，并设置`content: ' ' `,可令每一行都最少有一个空格。  
如果有强迫症，可以前后各加一个空格，让歌词居中。  
![歌词::after](https://of87cyikt.qnssl.com/blog/hexo/lrc-style.png)

### **【reduce】 获取当前播放的歌词，和下一个歌词**  
这是个人觉得正确使用reduce的一次了，虽然最后注释掉了。

```javascript
//写的时候挺费脑子的，不舍得删，注释掉吧。
let currentStamp = audio.currentTime
let { currentLrc, nextLrc } = lrc.filter(v => v[0]).reduce((prev, current) => {
  // 如果传入的值有了nextLrc，说明取到了想要的值
  // console.info(`reduce 得到的上一个的返回值：${JSON.stringify(prev)}`)
  let { currentLrc, nextLrc } = prev
  if (nextLrc) return prev
  return lrcTime2Second(current[0]) > currentStamp ? { currentLrc, nextLrc: current[0] } : { currentLrc: current[0], nextLrc: '' }
}, { currentLrc: '', nextLrc: '' })
```  
根据audio的当前时间，使用reduce取得当前歌词和下一条歌词。  
最初没有使用缓存变量时，简单的把上一条歌词取消激活用到了这个方法。虽然这种取消非常不严谨。但这个reduce写起来很爽。

## 真·END  

支持方法的切换，通过增加wrap函数和多处使用3元运算符，逗号运算符完成。//有点丧心病狂，就不截图了