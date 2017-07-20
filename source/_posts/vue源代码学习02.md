---
layout: post
title: vue源代码学习02
date: 2017-07-20 17:27:36
tags:
---
### Vue.prototype.$mount
$mount在Vue中是一个挂载方法,如果在实例化Vue到时候传入el参数就会自动将生成到实例挂载到对应到el上面,但是如果没有传入el参数就需要手动调用这个方法来挂载对象。
我打算从这个方法入手来看看挂载方法做了什么,首先挂载方法由两个部分组成,直接上代码。
``` JavaScript
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* Vue规定不能将组件挂载到body和html节点上 */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options
  // 获取template模板并且编译获得render方法
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
    if (template) {
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }

      const { render, staticRenderFns } = compileToFunctions(template, {
        shouldDecodeNewlines,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)
}

/*------------------runtime/index.js-----------------*/
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```
我们先看到`const { render, staticRenderFns } = compileToFunctions(template, {})`这行代码.他主要干了三件事情。
1. parser:解析template模板,获得AST树。
2. optimizer:标记静态节点。
3. codegen/code generate:生成渲染方法代码。
我们就这三个部分稍微展开来看一下。先来看看AST树，在vuejs中AST树由ASTNode组成，而ASTNode有三种类型ASTElement，ASTText和ASTExpression,下面是ASTElement的一些和解析有关到属性:
``` JavaScript
declare type ASTElement = {
  tag: string;
  attrsList: Array<{ name: string; value: string }>;  //属性列表
  parent: ASTElement | void; //父节点
  children: Array<ASTNode>;  //包含子ASTElement或者子ASTText和ASTExpression
  
  attrs?: Array<{ name: string; value: string }>;  //属性列表
  props?: Array<{ name: string; value: string }>;  //属性列表
  plain?: boolean;
  pre?: true;   //是否是不需要表达式的节点
  ns?: string;   //节点到命名空间

  component?: string;
  inlineTemplate?: true;
  slotName?: ?string;
  slotTarget?: ?string;
  slotScope?: ?string;
  scopedSlots?: { [name: string]: ASTElement };

  ref?: string;
  refInFor?: boolean;

  if?: string;
  ifProcessed?: boolean;
  elseif?: string;
  else?: true;
  ifConditions?: ASTIfConditions;

  for?: string;
  forProcessed?: boolean;
  key?: string;
  alias?: string;
  iterator1?: string;
  iterator2?: string;


  forbidden?: true;  //style和script是被禁止的。
  once?: true;
  onceProcessed?: boolean;

};

declare type ASTExpression = {
  type: 2;
  expression: string;
  text: string;
  static?: boolean;
};

declare type ASTText = {
  type: 3;
  text: string;
  static?: boolean;
  isComment?: boolean;
};
```
而一个template模板是怎么变成一颗AST树的,这跟很多到js静态代码解析大同小异。就是一大堆到正则解析。这里就不做深究了。

接下来就是optimizer了，他其实就做了一件事，就是标记静态节点,静态节点判断代码如下:
``` JavaScript
 if (node.type === 2) { // 不能是ASTExpression(AST表达式节点)
    return false
  }
  if (node.type === 3) { // 是ASTText(文本节点)
    return true
  }
  return !!(node.pre || (  //不需要表达式到节点
    !node.hasBindings && // 非动态绑定
    !node.if && !node.for && // 不是 v-if，v-for，v-else
    !isBuiltInTag(node.tag) && // 不是内置到tag
    isPlatformReservedTag(node.tag) && // 非组件
    !isDirectChildOfTemplateFor(node) &&
    Object.keys(node).every(isStaticKey)  //是否是静态key
  ))
``` 
标记静态节点到好处是可以在修补节点到时候跳过这些节点，另外在重新渲染到时候也不会去渲染他们。接下来就是传入之前生成的AST树生成编译完成的代码了。
他们会根据AST树到各个指令属性生成对应到方法,比如是否有for循环指令放入到对应到for循环方法中去让他可以在后续执行。他会两个方法一个是render方法另一个是staticRenderFns分别存放渲染方法和静态节点到渲染方法。
现在可以回过头来runtime/index.js里的$mount方法了,而他直接调用了`mountComponent(this, el, hydrating)`而在这个方法中做了这几件事情:
1. 调用之前生成到render方法获得VNode.
2. 利用生成的VNode更新需要装载到的节点，获得新的vm.$el。
当然这里面有很多复杂的判断,其中到一些重点到细节比如Vuejs的VDom模块，Watcher模块等等我会在后面深究。这里就借用官方的Vue生命周期图示在结束这一章的介绍，就当是抛砖引玉了。
[Vue生命周期]: https://cn.vuejs.org/images/lifecycle.png  "Vue生命周期"