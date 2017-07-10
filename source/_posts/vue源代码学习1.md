---
layout: '[layout]'
title: vue源代码学习1
date: 2017-07-06 14:39:07
tags:
---

忙了大半年之后总算有时间做一点自己想做的事情,之前在做技术选型的时候接触了vue,虽然没有在实际项目上使用这个前端框架。但是一直对这个前端框架很感兴趣。利用这个工作的空档打算好好的把这个框架学习一下。
学习一个像vue这类文档比较完善，活跃度也比较高的优秀开源项目时，我一般会遵循下面的顺序：
* 先查看官网的介绍，了解这个项目时基于什么目的开发的，主要为了解决什么问题，和其他相似同类型的库有什么区别。这里可以了解到项目开发的背景。在看具体实现的也可以知道开发者这么做的原因是什么。
* 然后我会去看package.json，从devDependencies中了解项目依赖了哪些库。会不会有什么特殊的语法，像vue就依赖了flow这个nodejs的库来做一些类型约束注释，那在了解vue源代码之前对flow也要进行一些学习。之前也写了一遍关于flow学习的文章：[flow学习]
* 接着就是看Contributing Guide，了解项目的目录结构，具体的开发环境，构建方式。这我觉得非常重要。
* 然后就开始看源代码了，而我一般会从每个index开始，尝试抓住每一块大概做了什么事情，然后在挑有兴趣或者重要的模块，挑细节的去看。我也比较喜欢看测试用例，在测试用例中有时候能很快的了解一个库的主线。

我就打算以这个顺序进行介绍。但是第一部分，vue的文档已经非常完备，包括在介绍中作者提到核心的代码主要把注意力集中在view展示层。从他和其他流行的前端库的比较中，我们也可以看见他们有各自相同和不同的地方。具体的细节读者可以自行阅读官方文档写的已经非常清楚。<https://vuejs.org/>
### 了解vue的文档结构
```
├── build      //包含构建依赖的配置文件。大多数情况下不需要动
├── dist       //包含了构建发布的文件。只能通过发布更新
├── flow       //包含Flow类型声明文件。这些声明都是全局的。通过.flowconfig中的[libs]指定
├── packages   //包含了vue-server-renderer,vue-template-compiler,weex-template-compiler,weex-vue-framework他们都是作为单独的npm模块。通过源代码自动生成
├── test       //包含所有的测试用例
├── src        //包含所有的原始代码
    ├── compiler   //包含模板渲染方法编译
    ├── core       //包含了平台兼容运行时代码
        ├── observer      // 监控相关代码，反应联动(reactivity)系统
        ├── vdom          // 虚拟Dom创建和补丁代码
        ├── instance      // 包含Vue实例构建方法和属性方法
        ├── global-api    //全局api
        ├── components    //通用抽象组件
    ├── server     //包含服务端渲染的代码
    ├── platforms   //包含指定平台代码
    ├── sfc         //包含单个文件解析组件逻辑。用于vue-template-compiler
    ├── shared      //包含通用工具代码
```
### 从构造方法开始
先来看一段代码
``` html
<div id="app">
  {{ message }}
</div>
```
``` JavaScript
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```
这个是作者提供的最简单的Vue使用的例子。我们就从这个就简单的例子入手一点点的分析Vue到底做了什么。首先先要找到Vue实例化组件的入口在哪里。让先我们结合package.json文档和build下面的build.js和alias.js看看vue.js是怎么被构建的。在package.json的scripts命令中有这么一行命令`"dev": "rollup -w -c build/config.js --environment TARGET:web-full-dev"`，就是这行命令在构建了vue.js。rollup是一个打包库他指定了一个配置文件build/config.js设置环境目标为web-full-dev。我们在build/config.js中找到web-full-dev，他是这个样子的：
``` JavaScript
'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  }
```
