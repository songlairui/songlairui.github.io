---
title: Vue SSR 官方文档实践·一：从零到粗暴混合前后端 
date: 2017-06-21 12:22:34
tags:
---
> Vue 2.3 发布很久了，距离第一次打开[ssr.vuejs](http://ssr.vuejs.org/en/)也很久了。  
现在我终于把其中的代码片段运行起来了。  

Github Repo: [https://github.com/songlairui/vue-playground](https://github.com/songlairui/vue-playground)

## 1. 尝试 `vue-server-renderer`  
实现使用 `console.log(html)` 将渲染过后的html打印到屏幕上。  
即，将 new Vue({...}) 变成输出结果。跳过从浏览器获得结果。  

## 2. 使用 import 、 export 关键字  
文档第四章[Source Code Structure](http://ssr.vuejs.org/en/structure.html) 出现了目录结构，而且js文件中有了import关键字。  
这里无法像前面的代码片段一样，直接在node里粘贴代码可执行。这里需要使用webpack。  

> 这时的我：使用vue-cli创建vue项目脚手架是唯一的webpack使用经验。  
> 然后我去 {% post_link learn-webpack 学习webpack %}

代码示例： [songlairui/vue-playground/demo/chapter4](https://github.com/songlairui/vue-playground/tree/master/demo/chapter4)  

**目标:** 让这样的目录能够执行

```shell
src
├── components
│   ├── Foo.vue
│   ├── Bar.vue
│   └── Baz.vue
├── App.vue
├── main.js # universal entry 我在实践时换了个名字
├── entry-client.js # runs in browser only
└── entry-server.js # runs on server only
```
main.js、entry-client.js、entry-server.js 代码从文档中复制来。  
*.vue 文件内容自己补充。

这里需要webpack的概念： 入口文件、输出文件、模块、插件、打包目标平台。  

#### 创建webpack配置文件  

> 啃了webpack文档后 <a href="{% post_path learn-webpack %}#基本配置文件">[跳转...]</a>  

**deal with 新出现的 import**

在基本的配置文件上增加`babel-loader`配置即可将import转译为node和浏览器可以支持引入方式了。  
这里需要将 `import` 转译为 `commonjs`方式，设置babel-loader的query为  
```javascript
... // 假装其他代码
module: {
  rules:[
    {
      test: /\.js$/,
      loader: 'babel-loader',
      query: [
        "presets": [
          "env", { "modules": "commonjs" }
        ]
      ]
    }
  ]
} 
... // 假装其他代码
```

modules 默认为commonjs，可以省略。其他可选umd,amd等。  
> // TODO 如果可能，会写一篇初试requirejs，commonjs.
  
**deal with vue单文件组件**  

`*.vue`单文件组件通过import引入，为其添加`vue-loader`,即可正确引入。

**module.target**  

webpack module默认的target是 web，为服务端代码进行打包时，需要设置target为 node 或 aysnc-node。  

## 3. 使用 VueRouter  

> 我已有的vue使用经验中，在router中，使用ensure即可轻松实现懒加载。  
 
动态引入模块，可以完成懒加载文件的打包
**改写router.js**
```javascript
// import About from './About.vue'
const About = () => import('./About.vue')
```
使用vscode中js formatter插件保存时自动格式化代码时，会将import的格式强行换行。可以停用自动格式化。  

**启用动态加载**
webpack配置rules中，为babel-loader启用动态引入插件
```javascript
...
{
  test: /\.js$/,
  loader: 'babel-loader',
  query: {
    presets: [
      ['env', { modules: false }]
    ],
    plugins: [
      "syntax-dynamic-import"
    ]
  }
}
...
```

dist目录下打包结果如下：  

| client.conf | - | server.conf |
| ------| ------ | ------ |
| 0.js | - | 0.js |
| client.bundle.js | - | server.bundle.js |  

client和server配置文件都会生成0.js,1.js.... 
在未使用manifest情况下，两个配置文件生成内容不一样（server配置文件中 module.target为’node‘）。  
如果同时使用，需要调整输出位置，避免代码覆盖。

## 4. 粗暴的融合前后端

我对需求服务端渲染的理解是，使用爬虫获取指定url时，得到的是该url渲染好的html内容。  
在浏览器上第一次打开此页面时，请求到的html内容是已渲染好的。  
在浏览器上进行后续的交互时，用的是vue框架的交互逻辑，而不重新发起url请求。  

**客户端使用场景**：  
客户端build之后，使用方式是在一个body只包含`div#app`元素的index.html底部注入`script:src`引入build好的client.bundle.js。  
然后使用http server提供对这个index.html的访问。

**服务端使用场景**：  
在server.js中 配置express，并require 打包好的entry-server.js，填充逻辑。  
然后运行`node server.js`, 就能像使用nginx 托管静态html文件夹那样按照url访问指定html页面。

现在，在App.vue文件中根元素上添加 `id='app'`, 渲染的到的html中包含此 `id='app'`. 对此#app挂载Vue即常规的Vue使用方法。  
在2.3版本中对服务端渲染进行了支持，会自动辨识服务端渲染好的dom。  
 
使用服务端生成的内容，替换客户端使用场景中index.html 就是简易的融合前后端。  
即，在服务端所使用的index.template.html页面中，添加 `script:src` 引入客户端build的client.bundle.js。

**融合细节**：  
1. 区分服务端与客户端打包代码  
为 server.conf, client.conf  设置不同的output.path  
2. 修改index.template.html  
添加 `<script src="./dist/bundle.js"></script>`  
3. server.js中为express添加中间件处理逻辑  
请求0.js时，会直接请求根目录，这里redirect一下，让它取到正确的0.js  

```javascript
app.use(function(req,res,next){
  // 使用bunlde懒加载0.js时，引用路径是不正确的，我不知道怎么配置，这儿强行改一下
  if(/\.js$/.test(req.url)){
    res.redirect('/dist' + req.url)
  } else {
    next()
  }
})
```


代码示例： [songlairui/vue-playground/demo/chapter5](https://github.com/songlairui/vue-playground/tree/master/demo/chapter5)  


## END  
按照个人理解，成功使用了下服务端渲染的结果。  
过程虽然很粗糙，也是使出了浑身解数的。  