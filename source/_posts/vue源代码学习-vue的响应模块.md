---
layout: post
title: vue源代码学习-vue的响应式系统模块
date: 2017-07-29 12:30:59
tags:
---

### 什么是响应式系统

我们之前已经了解了Virtual Dom是怎么一回事，而在vue和其他前端框架中都有一个很重要的概念就是响应式系统，不论是使用脏检查,依赖收集,还是其他方式都是为了一个目的，就是在数据发生变化的时候通知
所相关的组件进行更新,那么我们来看看在vue中,这个模块是如何工作的。

### Object.defineProperty

在我们了解vue的响应式系统之前我们先要了解一下Object.defineProperty,他是ES5.1 规范中提出的,我们可以通过他来定义(劫持)对象的读取和写入方法,像这样:
``` JavaScript
Object.defineProperty(_object, 'p', {
  enumerable: true,
  configurable: true,
  get() {          
   ...
  },
  set(newValue) {   
   ...
  }
})
``` 
他的中文介绍在这里[MDN-Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)。
而在vue的响应系统中就用到这个方法,这也是他不支持IE8及以下版本浏览器的原因。

### vue中的响应系统
我们了解了Object.defineProperty之后还要搞清楚一个关系,在vue中响应模块主要依赖三个部分展开的,他们分别是Observer,Dep,watcher.
Observer:负责监控属性变化。
Dep:负责收集管理观察者。
watcher:观察者,响应变化。
他们三个的关系是这样的:
![](http://image.jiantuku.com/17-7-29/79326618.jpg?attname=file_1501311348782_15964.png&e=1501311610&token=el7kgPgYzpJoB23jrChWJ2gV3HpRl0VCzFn8rKKv:MUaLIpJNPAcSZAjbXIwQgU8M-b8=)
在来看一下Observer:
``` JavaScript
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that has this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

 
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i], obj[keys[i]])
    }
  }

 
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```
而defineReactive方法就是使用了Object.defineProperty劫持属性get/set的方法了:
``` JavaScript
function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
        }
        if (Array.isArray(value)) {
          dependArray(value)
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
``` 
当每个属性调用defineReactive方法的时候，每个属性都有各自的Dep对象，他们收集了依赖这个属性的watcher对象，当对象set数据发生改变的时候，调用对应dep对象的notify通知他们发生改变。
另外他们还共享同一个Dep对象的target,他存储了当前的watcher对象。来看我们之前熟悉的一段代码`vm._watcher = new Watcher(vm, updateComponent, noop)`。另外结合wathcer对象的构造函数(wathcer对象同时也用于$watch方法)。
``` JavaScript
 constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: Object
  ) {
    this.vm = vm
    vm._watchers.push(this)
    // options
    if (options) {
      this.deep = !!options.deep
      this.user = !!options.user
      this.lazy = !!options.lazy
      this.sync = !!options.sync
    } else {
      this.deep = this.user = this.lazy = this.sync = false
    }
    this.cb = cb
    this.id = ++uid
    this.active = true
    this.dirty = this.lazy 
    this.deps = []
    this.newDeps = []
    this.depIds = new Set()
    this.newDepIds = new Set()
    this.expression = process.env.NODE_ENV !== 'production'
      ? expOrFn.toString()
      : ''
    
    if (typeof expOrFn === 'function') {
      this.getter = expOrFn
    } else {
    //当expOrFn为非function对象,则返回用于获取vm对象属性的方法。
    //例如:expOrFn：'a'    则返回用于获得vm.a对象属性的方法
      this.getter = parsePath(expOrFn)                                                                      
      if (!this.getter) {
        this.getter = function () {}
      }
    }
    this.value = this.lazy
      ? undefined
      : this.get()
  }
  
  ------------------------------get方法------------------------
  
   get () {
      //设置当前Dep.target对象为目前的watcher。
      pushTarget(this)
      let value
      const vm = this.vm
      try {
        //调用之前的expOrFn方法获取返回值
        //如果是updateComponent或者其他获取对象的方法,
        //势必可能会依赖别的属性来生成视图或者对象。那么就会调用对应依赖属性的get方法
        //这个时候当前的Dep.target对象已经指向当前的watcher
        //而当前watcher对象将会被加入到其依赖属性的dep对象中。在这个属性发生改变时通知watcher
        value = this.getter.call(vm, vm)
      } catch (e) {
        if (this.user) {
          handleError(e, vm, `getter for watcher "${this.expression}"`)
        } else {
          throw e
        }
      } finally {
        // 深度依赖收集，
        // 深层级的对象如:a.b.c.e。设置deep选项就逐级收集watcher对象到当前Dep.target
        if (this.deep) {
          traverse(value)
        }
        //弹出当前target
        popTarget()
        this.cleanupDeps()
      }
      return value
    }
```
借用一下[https://github.com/jin5354/404forest/issues/60](https://github.com/jin5354/404forest/issues/60)中的图:![](https://camo.githubusercontent.com/b76d07b52d5b7c6b313663ef3a06906a04c96acb/68747470733a2f2f7777772e343034666f726573742e636f6d2f696d67732f626c6f672f726561637469766974792d73797374656d2d332e706e67)
那么这里依赖就收集完毕了。当属性发生改变时在set方法中就会执行对应的`dep.notify()`方法通知依赖这个属性的watcher进行update。而update方法是这样的:
``` JavaScript
update () {
    //如果是懒更新则标记为脏数据，表示数据已经发生改变，在计算数据的时候获得最新值。
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      //异步更新直接调用run方法
      this.run()
    } else {
    //否则加入watcher队列
      queueWatcher(this)
    }
  }
```
在`queueWatcher(this)`中方法会调用`nextTick(flushSchedulerQueue)`，在这个方法里有一些兼容性代码,判断当前平台是否支持Promise或者MutationObserver如果都不支持的话则调用`setTimeout(nextTickHandler, 0)`setTimeout 0原理和干了什么可以看[这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)
下面附上flushSchedulerQueue方法:
``` JavaScript
function flushSchedulerQueue () {
  flushing = true
  let watcher, id

  
  queue.sort((a, b) => a.id - b.id)

  
  for (index = 0; index < queue.length; index++) {
    watcher = queue[index]
    id = watcher.id
    has[id] = null
    //调用watcher update
    watcher.run()
    if (process.env.NODE_ENV !== 'production' && has[id] != null) {
      circular[id] = (circular[id] || 0) + 1
      if (circular[id] > MAX_UPDATE_COUNT) {
        warn(
          'You may have an infinite update loop ' + (
            watcher.user
              ? `in watcher with expression "${watcher.expression}"`
              : `in a component render function.`
          ),
          watcher.vm
        )
        break
      }
    }
  }

  const activatedQueue = activatedChildren.slice()
  const updatedQueue = queue.slice()

  resetSchedulerState()

  callActivatedHooks(activatedQueue)
  callUpdatedHooks(updatedQueue)

}
```
在watcher.run中就会调用getter方法获取最新的值(重新渲染组件),，并且调用值发生变化以后的回调函数:
``` javascript
run () {
    if (this.active) {
      //调用get获得最新的值
      const value = this.get()
      if (
        value !== this.value ||
        // Deep watchers and watchers on Object/Arrays should fire even
        // when the value is the same, because the value may
        // have mutated.
        isObject(value) ||
        this.deep
      ) {
        // set new value
        const oldValue = this.value
        this.value = value
        if (this.user) {
          try {
            this.cb.call(this.vm, value, oldValue)
          } catch (e) {
            handleError(e, this.vm, `callback for watcher "${this.expression}"`)
          }
        } else {
          this.cb.call(this.vm, value, oldValue)
        }
      }
    }
  }
```

