---
title: Mocha 小试，解析 INSERT VALUES 
date: 2017-07-29 10:44:35
tags:
---

> 项目使用DB2数据库，上线时要将更新的SQL打包。统计SQL时，同样的数据两次INSERT会提示主键冲突。于是，尝试写一个分析SQL语句的功能，用来替代或简便手工检查。  
从npmjs.org上，找其他包时，发现其包内的单元测试。决定尝试一些这样的套路。  
<!--more-->
# 解析 VALUES    
在 npmjs.org 上选了若干包安装尝试解析项目中的SQL语句，遇到了问题：VALUES中含有单引号转义时，解析不成功，或者拿到结果需要再解析一遍单引号转义。  

观察VALUES的结构，手动解析逻辑比较简单。  

## TRY 1. 尝试使用正则  

VALUES中划分每个字段的标示是逗号，当value中有逗号时，一行正则无法区分是value中的逗号，还是外部的逗号。  
将划分标示扩展为 \` ' , '\` 依然会遇到标识在value中出现的可能。 
尝试先解析转移的单引号 ，未能区分value两侧的单引号，还是转义的单引号。  
尝试写了一些正则，代码没成功，丢弃了。  

## TRY 2. 逐个字符分析  

分析按照大脑识别的过程，当遇到单引号时，根据之前的状态，是准备取字段，还是正在取字段中的值确定单引号用来结束或开始转义，还是结束或开始取值。  

于是,**约定状态**  

```javascript
  let mode = {
    WILLINGCOLLECT: 1, // 准备收集value，默认起点。遇到null或者
    COLLECTING: 2, // 正在收集数据，
    PREPARETOCHANGE: 3, // 收集数据中，遇到了单引号，准备发生变化
    WILLINGSPLIT: 4 // 准备遇到分割符
  }
```
在不同状态下，读到的特殊符号，完成转义或结束取值行为，并更改到新状态。  
状态发生变化时，是在后一个字符确定的，所有识别到变化时候，需要 i-- 或者手动补上上一位的操作。

然后,**收集错误**  

解析完毕之后，临时取值池中的值应该为undefined，如果不为空，则判定为异常停止   
遇到不预期的字符，可以跳出for循环，设置标示变量，ifBreakFor，用来在switch中跳出外部for  

# 使用 mocha + chai 

代码： [mocha 测试 VALUES 解析](https://github.com/songlairui/js-sql-parser-test/blob/master/test/VALUES.spec.js#L17)  

直接阅读mocha文档，不觉得有什么用途。  
当写了这个解决其他包不能够的痛点时，用mocha让测试意图非常清晰+固定。  

执行结果 
```bash
  err RETURN
    ✓ should return null in result.err, and values in values

  VALUES in some conditions
    一般的VALUES : VALUES('value1')
      ✓ should return `value1`
    VALUES为null : VALUES(null)
      ✓ should return `null`
    中间含有一个单引号转义: VALUES('_''_')
      ✓ should return `_'_`
    开头含有一个单引号转义: VALUES('''_')
      ✓ should return `'_`
    结尾含有一个单引号转义: VALUES('_''')
      ✓ should return `_'`
    结尾含有多个单引号转义: VALUES('''''_''''')
      ✓ should return `''_''`
    有单引号转义，且含形如 ' , ' 的字符片段: VALUES(''' , ''')
 此种情况正则难以区分
      ✓ should return `' , '`
    有单引号转义，且含形如  ' ,null , ' 的字符片段: VALUES(''' , null , ''')
此种情况正则难以区分
      ✓ should return `' , null , '`
    单引号转义前后有空格: VALUES(''' a ''')
      ✓ should return `' a '`
  10 passing (10ms)

```

这是一个感受到mocha必要的地方。