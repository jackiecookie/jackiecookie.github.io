---
layout: post
title: Virtual Dom以及vuejs中的Virtual Dom
date: 2017-07-22 11:45:34
tags:
---

### 什么是Dom

在介绍虚拟Dom之前,我们先简单了解一下什么是Dom.Dom是Document Object Model的简称。Dom是一种通过对象来表示文档结构的方式。我们可以通过Dom来创建结构，
新增修改删除节点和内容。在浏览器端我们可以使用Javascript和Css来操作Dom。也就是说Dom是一个我们如何获得修改增加删除Html节点的规范。

### 原始Dom有什么问题？

在Html Dom中,我们一个节点对象都代表一个Html节点。在现代到前端场景中,类似ASP或者一些Web App的应用我们可能需要使用大量到元素来构建一个页面，成千上万的节点就会有成千上万到节点对象(Dom),
你会有一个非常大到Dom树,我们可能会有很多种情况需要修改增加删除节点。

### 为什么原始Dom会慢?

其实更新Dom并不慢，他就像更新一个js对象，但是了解浏览器渲染过程的应该知道，渲染引擎负责解析HTML来创建Dom，也负责解析css，在HTML上应用css来构建渲染树，而Layout过程
给了每个渲染树中的节点一个纵坐标,表示了这个节点在哪里渲染和显示。在我们修改增加删除节点时候,特别是节点层级比较多的时候或者节点比较多的时候，重新计算css和修改layout
使用了复杂的算法从而影响了性能，因此更新原始Dom不仅仅是更新了Dom，还牵涉了很多过程，每次更新原始Dom都会重复这些过程，这就是为什么Dom慢的原因，
而Virtual Dom就是尽可能到减少这个过程的时间。

### 什么是Virtual Dom

Virtual Dom是HTML DOM抽象出来的结果，你也可以理解为HTML DOM是HTML DOM简化以后到结果,他允许我们在一个虚拟的Dom世界里避免"真正"到Dom操作。

### ReactJS的Virtual Dom简述

最早听到Virtual Dom是在ReactJS中，而Virtual Dom的核心步骤主要是三个
1. 构建Virtual Dom。
2. 两个Virtual Dom比较差异。
3. patch老到Virtual Dom。
而ReactJS中Virtual Dom之所以快是因为他使用了:
1. 高效的diff算法。
2. 批量更新操作。
3. 只更新需要更新的子节点。
4. 使用观察模式而不是脏检查来监控节点更新。
网上已经有许多的ReactJS的Virtual Dom实现的文章，大家可以找来看看。有时间我也会深究一下。

### vue中Virtual Dom的实现

首先在vue中有部分到vdom实现是基于[Snabbdom](https://github.com/snabbdom/snabbdom)库而实现的。在之前的vue源代码学习02中我们已经知道了Vue根据`const { render, staticRenderFns } = compileToFunctions(template, {})`
获得了render方法.我们先来看看生成完的render方法。来看一个最简单的例子：
``` JavaScript
const vm = new Vue({
      data: {
        message: 'Hello Vue!'
      },
      template: '<div>{{ message }}</div>',

    }).$mount()
``` 
他生成到render方法是这个样子的：
``` JavaScript
(function() {
with(this){return _c('div',[_v(_s(message))])}
})

/*------------------我们之前也已经知道-----------------*/
vm._c = (a, b, c, d) => createElement(vm, a, b, c, d, false)
Vue.prototype._v = function createTextVNode (val: string | number) {}
Vue.prototype._s = toString

```
而VNode是这样定义的:
``` JavaScript
export default class VNode {
  tag: string | void;
  data: VNodeData | void;
  children: ?Array<VNode>;
  text: string | void;
  elm: Node | void;
  ns: string | void;
  context: Component | void; 
  functionalContext: Component | void; 
  key: string | number | void;
  componentOptions: VNodeComponentOptions | void;
  componentInstance: Component | void; 
  parent: VNode | void; 
  raw: boolean;
  isStatic: boolean; 
  isRootInsert: boolean; 
  isComment: boolean; 
  isCloned: boolean; 
  isOnce: boolean;
  asyncFactory: Function | void; 
  asyncMeta: Object | void;
  isAsyncPlaceholder: boolean;
  ssrContext: Object | void;
}
```
再看一下最主要的createElement方法:
``` JavaScript
function _createElement (
  context: Component,
  tag?: string | Class<Component> | Function | Object,
  data?: VNodeData,
  children?: any,
  normalizationType?: number
): VNode {
  //避免使用已经被监控的data对象作为vnode对象
  if (isDef(data) && isDef((data: any).__ob__)) {
    process.env.NODE_ENV !== 'production' && warn(
      `Avoid using observed data object as vnode data: ${JSON.stringify(data)}\n` +
      'Always create fresh vnode data objects in each render!',
      context
    )
    return createEmptyVNode()
  }
  //  <component :is="view"></component>
  if (isDef(data) && isDef(data.is)) {
    tag = data.is
  }
  if (!tag) {
    // 没有tag返回空VNode对象
    return createEmptyVNode()
  }
  // 支持单个子方法作为默认的scopedSlots：https://vuejs.org/v2/guide/components.html#Scoped-Slots
  if (Array.isArray(children) &&
    typeof children[0] === 'function'
  ) {
    data = data || {}
    data.scopedSlots = { default: children[0] }
    children.length = 0
  }
  //标准化childrens
  if (normalizationType === ALWAYS_NORMALIZE) {
    children = normalizeChildren(children)
  } else if (normalizationType === SIMPLE_NORMALIZE) {
    children = simpleNormalizeChildren(children)
  }
  let vnode, ns
  if (typeof tag === 'string') {
    let Ctor
    ns = config.getTagNamespace(tag)
    //是否是正常到Tag
    if (config.isReservedTag(tag)) {
      // 根据平台生成对应tag的VNode
      vnode = new VNode(
        config.parsePlatformTagName(tag), data, children,
        undefined, undefined, context
      )
      //option中是否已经定义了这个component
    } else if (isDef(Ctor = resolveAsset(context.$options, 'components', tag))) {
      // 创建component
      vnode = createComponent(Ctor, data, context, children, tag)
    } else {
      vnode = new VNode(
        tag, data, children,
        undefined, undefined, context
      )
    }
  } else {
    // 直接创建component
    vnode = createComponent(tag, data, context, children)
  }
  if (isDef(vnode)) {
    if (ns) applyNS(vnode, ns)
    return vnode
  } else {
    return createEmptyVNode()
  }
}
``` 
` _c('div',[_v(_s(message))])` 这个render方法返回的VNode主要属性是下面这个样子的:

``` JavaScript
vnode:VNode{
 context:Vue,
 tag:'div',
 children:[{
    text:'{{ message }}'
 }]
}
``` 

接下来我们回到mountComponent方法
``` JavaScript
updateComponent = () => {
    vm._update(vm._render(), hydrating)
}
vm._watcher = new Watcher(vm, updateComponent, noop)
```
这里的Watcher我们先简单的介绍一下,当代码执行到时候updateComponent会作为获取值的表达式来获得value,当vm对象到任何属性改变的时候都会重新调用这个表达式来重新获取值。
这个方法很关键，我们后面在展开。先看更关键vm._update方法，前面我们已经获得了_render方法返回到VNode。来看看_update方法
``` JavaScript
Vue.prototype._update = function (vnode: VNode, hydrating?: boolean) {
    const vm: Component = this
    if (vm._isMounted) {
      callHook(vm, 'beforeUpdate')
    }
    //获得之前的节点，oldEl，oldVNode
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const prevActiveInstance = activeInstance
    activeInstance = vm
    //把当前传入的vnode给变量_vnode,作为下次匹配的oldVnode
    vm._vnode = vnode
    //如果之前没有vnode
    if (!prevVnode) {
      // initial render
      vm.$el = vm.__patch__(
        vm.$el, vnode, hydrating, false /* removeOnly */,
        vm.$options._parentElm,
        vm.$options._refElm
      )
      vm.$options._parentElm = vm.$options._refElm = null
    } else {
      //如果之前有vnode则需要更新
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    activeInstance = prevActiveInstance
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
  }
```
我们终于看到了__patch__方法。在vuejs中patch方法中同时也实现了diff算法。这个方法主要做了这几件事:
1. 如果没有老元素，则直接创建新元素。
2. 如果有老元素,老元素是否为原始Element，是原始Element考虑是否是服务器渲染([SSR](https://ssr.vuejs.org/))的，服务器渲染则混合(hydrate)。
3. 如果有老元素,老元素不是原始Element，比较老元素和新元素并且更新元素。

服务器渲染混合的情况我们先放一边，直接来看patchVNode方法
![avatar](http://image.jiantuku.com/17-7-26/82308669.jpg?attname=file_1501061830126_61f0.png&e=1501070410&token=el7kgPgYzpJoB23jrChWJ2gV3HpRl0VCzFn8rKKv:ORMmnZP-YN6RW5i1G7xQe68xZBg=)

