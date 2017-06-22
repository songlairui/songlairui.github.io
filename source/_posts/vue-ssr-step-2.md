---
title: Vue SSR 官方文档实践·二：前后端混合从粗暴到正常
date: 2017-06-22 13:12:43
tags:
---
> 上一篇{% post_link vue-ssr-step-1 %}之后，运行[Build Configuration](http://ssr.vuejs.org/en/build-config.html) 章节示例代码，比较快了。  
> 不过多加了几个webpack选项，混合起来变得更加简单。大神铺路铺得就是好。。  

**实践Target:** [Build Configuration](http://ssr.vuejs.org/en/build-config.html)
<!--more-->
### Server端 启用vue-ssr plugin  

| 启用前| - | 启用后 |
| ------| ------ | ------ |
| ![server1.png](https://ooo.0o0.ooo/2017/06/22/594b584a3e1e8.png) | - | ![server2.png](https://ooo.0o0.ooo/2017/06/22/594b5849e0aad.png) |
| 启用前，异步加载的组件，被分割开来| - | 启用后Server端打包不再进行分割，只输出一个文件 |    

客户端打包，对文件分割实现懒加载，可以令浏览器只加载需要的内容，有体验提升。  
但服务端打包的文件，给后端程序使用，不通过网络请求获取，分割代码意义不大甚至降低性能，所以打包成一个文件更好。

### Client端 启用 manifest，启用vue-ssr plugin

**相关配置文件**  
```javascript
plugins: [
    new webpack.optimize.CommonsChunkPlugin({
      name: 'mainfest',
      minChunks: Infinity
    }),
    new VueSSRClientPlugin()
  ]
```
**对比** 

| 启用前| - | 启用后 |
| ------| ------ | ------ |
| ![client1.png](https://ooo.0o0.ooo/2017/06/22/594b5c09924cb.png) | - | ![client2.png](https://ooo.0o0.ooo/2017/06/22/594b5c0a7353b.png) |
| 启用前，前后端输出文件名称一致，会造成覆盖| - | 启用后，生成 manifest, 可以使用vue-ssr插件自动注入js |  

启用前，在页面注入JS，需要手动在 index.template.html 作如下添加 
```html
<body>
  <!--vue-ssr-outlet-->
  <script src="./dist/bundle.js"></script> <!-- 手动添加，启用后不需要添加这一行-->
</body>
```

## 修改server.js  

**1. 更改renderer**  

| 前| - | 后 |
| ------| ------ | ------ |
| ![render1.png](https://ooo.0o0.ooo/2017/06/22/594b6c340b19b.png) | - | ![render2.png](https://ooo.0o0.ooo/2017/06/22/594b6c375263e.png) |
| createRender | - | 更改为 createBundleRenderer，并由打包生成的单文件创建 |  

**2. 更改express配置** 

| 前| - | 后 |
| ------| ------ | ------ |
| ![app1.png](https://ooo.0o0.ooo/2017/06/22/594b6e36a7ecf.png) | - | ![app2.png](https://ooo.0o0.ooo/2017/06/22/594b6e3622523.png) |
| 从打包得到的文件引入createApp | - | render中自动包含app内容，不需要引入。只需要传入context |  

## 更改路径，撤销粗暴  

**添加webpack关键配置**
```javascript
output:{
  publicPath: '/dist/',
  ...
}
```
自动注入的script.src 都指向 dist 目录下文件，server.js中强制redirect的中间件逻辑可以删除了。


## END  

回顾一下，完成服务端渲染的配置，大部分靠webpack的熟练度。  

代码地址： [songlairui/vue-playground/demo/chapter5](https://github.com/songlairui/vue-playground/tree/master/demo/chapter5)  
```shell
├── webpack.base.conf.js     # 创建baseConfig，方便使用webpack-merge
├── webpack.client2.conf.js  # 启用vue-ssr-client-bundle 插件，这是个子插件。启用manifest插件。
├── webpack.server2.conf.js  # 启用vue-ssr-server-plugin 插件，这是个子插件。打包只出一个文件。
├── server2.js               # 简化 server.js 逻辑。去掉强制redirect逻辑。
``` 