---
title: Webpack 学习记录
date: 2017-06-18 13:58:32
layout: post
tags:
---
>  我想要从零开始构建自己的应用，比如使用ssr-vue。而官方vue-cli没有类似原有template的ssr版。  
>  看来，我需要学会webpack了  

#### 已有的学习经历   
vue-cli自带的webpack配置，看过好几次。自己添加一些chunk，自动生成多个页面，觉得自己做了很大一件事情，而客观上并不大。  
webpack文档尝试看了好几次，从来觉得太长没看完。  
<!--more-->
### Current  

链接地址：  [http://www.css88.com/doc/webpack2/concepts/module-resolution/](http://www.css88.com/doc/webpack2/concepts/module-resolution/)  
我还是从头开始看了入口文件，配置方法。  
目前看来，概念部分，应该是一口气看完的。然后再在指南不烦，联系代码片段。  
这儿，我联想到了vimtutor中的练习方式。

### 基本配置文件  

> 啃了webpack文档后，能快速手打webpack的最简单配置文件。所有的配置都在此基础上添加。  

```javascript
module.exports = {
  entry: '/path/to/entry/file',
  output: {
    filename: 'output.js',
    path: __dirname
  },
  module:{
    rules:[
      ...
    ]
  }
}
```

**常见选项**  

1. resolve  
webpack 解析。
``` javascript
resolve: {
    alias: {
      'vue': 'vue/dist/vue.common.js'
    }
    // extensions: ['.ts', '.vue', '.js']
  }
```
默认逻辑不够准确时，需要手动添加alias。  
需要为vue使用 vue.esm.js时，可以通过添加alias更改。  

2. devtool  
``` javascript  
  devtool: 'source-map'
```
用来生成source-map文件  

3. entry  
可由基本的 值 变成 键-值。  
```javascript
entry: {
  main: './main.js',
  server: './server.js'
}
```

4. output  
```javascript
output: {
  filename: '[chunk].bundle.js',
  path: path.resolve(__dirname, 'dist')
}
```  
需要先 `const path = require('path')`, 这里path属性的写法即使用兼容全平台的方式指定当前目录下的dist目录为输出目录。  
`filename`属性的`[chunk]`能让entry中的main.js 输出为 main.bundle.js ，类似的还有`[name]`、`[chunkhash]`、`[hash]`。  

5. rules  
指定loader。webpack2兼容webpack1等loader写法。
