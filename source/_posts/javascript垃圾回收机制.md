---
layout: post
title: javascript垃圾回收机制
date: 2019-07-01 10:53:41
tags:
---

## 前言

其实大多数的时候作为javascript开发者不需要太关心内存的使用和释放,因为所有的javascript环境都实现了各自的垃圾回收机制(garbage collector(GC)),但是随着现在的SPA越来越多也越来越大,越来越追求极致的性能渐渐也要求开发者能够适当的了解一些垃圾回收机制内部的实现原理,在性能优化和追踪内存泄漏的时候都能够起到一点帮助。看一段内存泄漏的代码
``` javacript

var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var c = 'a'
  function unused() {
    if (originalThing) {
      console.log("hi");
    }
  };

  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log('1111');
    }
  };
};

setInterval(replaceThing,1000)

```
最早想要去深入了解javacript GC是看到这道找内存泄漏的题目(具体怎么内存泄漏,我们后面在分析).任何一种GC管理都需要做这几步:
1. 识别哪些对象需要被回收。
2. 回收/重复使用需要被回收对象的内存。
3. 压缩/整理内存(有些可能没有)

而常见的识别对象是否需要回收的机制有下面几种:
* 引用计数 (Python)
* 逃逸分析 (Java)
* Tracing/Reachable 追踪分析 (javascript)

今天就主要看一下V8中GC的具体实现方式

## Tracing/Reachable 追踪分析
GC的第一步就是要找出哪些对象需要被回收,哪些不需要。在追踪分析(Tracing/Reachable)中,认为可以被追踪到(reachability)的对象认为是不能被回收的对象,剩下的不能被追踪到的对象就是要回收的对象。
在V8中,追踪分析会从根对象开始(GC Root)根据指针将所有被能被追踪到的对象标记为reachable,javascript中根对象包括调用堆栈和global对象。


## The Generational Hypothesis
Generational Hypothesis的意思是大部分的对象在早期就需要被回收。基于这样的一个假设,有很多的编程语言的垃圾回收机制在设计的时候都是将内存分代,年轻代(young generation),和老代(old generation)。
这里的代其实就是开辟两块space分别存储刚被分配的对象和经过两次GC还是没有被回收的对象。在V8中有两个垃圾回收器分别对年轻代和老代进行垃圾回收,Scavenger针对年轻代进行垃圾回收,Major GC针对老代进行垃圾回收,他们的算法也是不同的。

### Scavenger
V8在年轻代的内存space使用的是semi-space算法,也就是说将内存分为两半,同时只有一块的内存能被使用,另外一半是完全空的(或者说这一半内存都是可以被分配的)。在程序开始执行的时候,将所有的变量都分配可以被使用的一半内存中(叫做from-space)。当第一次GC开始的时候根据追踪分析结果,将所有可以reachable的对象(不能被释放的对象),全部转移到剩余一半可以被分配的内存中(to-space)，这样from-space中的内存又全部可以被分配了，这个时候如果又有新申明的对象需要分配内存,就会分配到这一块内存当中了,最后在转移完不能被释放的对象之后,还需要更新引用指针,指向在to-space中最新的地址。

![第一次GC](http://img.pandihai.com/03.svg)<center><font color=gray size=2>第一次GC</font></center>


第二次GC开始的时候,在原本的to-space中仍然不能被释放的对象首先转移到老代(old generation)的space中,这时候to-space中又全部可以被分配,重复之前的操作。从from-space中将不能被释放的对象转移过来。完成2次GC之后,存货了两次的对象现在就在老代里面了,而存活一次GC的对象现在就在to-space中了,这个to-space也被叫做intermediate generation(中生代).在Scavenger中回收内存有三个过程:标记(追踪分析),转移(from-space to to-space),更新指针地址。

![第二次GC](http://img.pandihai.com/04.svg)<center><font color=gray size=2>第二次GC</font></center>


在这种内存回收的机制中,其中一个问题就是转移对象的时候是会消耗一定性能的,但是根据Generational Hypothesis的假设大部分的对象在早期就会被回收了,这也就意味着只有少部分不能被回收的对象需要被移动，这也意味着如果这个假设不成立，比如我们的代码中有很多的闭包导致很多的作用域不能被释放,那么将会有大量的对象需要在space之间转移,是比较浪费性能的。但是相反的,基于大部分对象都可以在早期被回收的假设,如果大部分的对象在早期就可以被释放,这种机制的内存回收对这需要在早期就回收的对象其实是什么都不需要做的,只需要把不能释放的少部分对象进行转移（from-space to to-space）,然后在下次分配内存的时候把这部分需要释放的对象所占的内存直接覆盖就可以了(rewrite dead object)。

#### Parallel
Parallel是V8中调度线程进行内存回收的一种算法,指的是主线程和帮助线程同时进行相同工作量的内存回收,这种算法还是会停止主线程正在进行的全部工作,但是工作量被平摊到几个线程之后,理论上时间也被参与线程的数量整除了(加上一些同步调度的开销)。Scavenger就是使用的这种线程调度机制,当需要进行内存回收的时候,所有的线程获得一定数量的存活的对象引用指针,开始同时将这些存活对象搬运到to-space中。不同的线程可能通过不同引用路径访问到同一个对象,当线程将存活对象转移到to-space之后,更新完指针地址后,会在from-space的老对象中留下一个forwarding指针,这样其他线程找到这个对象之后就可以通过这个指针来找到新的地址更新引用地址了。

![Scavenger平行调度](http://img.pandihai.com/05.svg)<center><font color=gray size=2>Scavenger平行调度,同时有多个帮助线程和主线程参与</font></center>

### Major GC
Major GC主要负责老代的内存回收,同样也是三个过程:标记(追踪分析),清除,整理压缩内存。标记这一步和Scavenger一样通过追踪分析确定哪些内存需要被回收,然后在对象被回收以后将被回收的内存加入到free-list这个数据结构中,free-list就像是一个个抽屉,每个抽屉的大小代表了从这个地址开始可以被连续分配的内存的大小,当我们需要在老代中重新分配内存的时候就可以快速的根据需要分配内存的大小找到一个合适的抽屉把内存进行分配。最后就是进行内存整理,这个就好像是Windows系统整理磁盘一样,将还没被幸存的对象利用free-list查找拷贝到其他的已经被整理完的page中,这样使小块的内存碎片也被整理完之后加以利用。跟Scavenger中一样来回拷贝对象也会有性能的消耗,在V8中只会对高度碎片化的page进行整理,对其他的page进行清除,这样在转移的时候也是一样的只需要转移存活的对象就可以了。

#### Concurrent
Concurrent同样也是V8中进行内存回收的线程调度算法,当主线程执行Javascript的时候,帮助线程同步进行内存回收的一些工作。相比Parallel来说这个算法要复杂的多,可能前一毫秒帮助线程在进行GC操作,后一毫秒主线程就改变了这个对象。也有可能帮助线程和主线程同时读取修改同一个对象。但是这种算法的优势就是当帮助线程进行GC工作的时候,主线程可以继续执行JavaScript,完全不会受到影响。Major GC就是采用的这个算法,当老代的内存到达一定系统自动计算的阀值,就开始进行Major GC,首先每个帮助线程都会获得一定数量的对象指针,开始针对这些对象进行标记,然后根据对象的引用指针对reachable对象都进行标记,在进行这些标记的同时,主线程仍然在执行JavaScript没有受到影响。当帮助线程完成标记,或者老代触及了设定的阀值,主线程也开始参与GC,他首先进行一步快速的标记确认,确保帮助线程在标记的同时主线程修改的对象标记正确(在帮助线程进行标记的时候,如果主线程执行的JavaScript修改了对象会有Write barriers,类似于有个标记)。当主线程确认所有存活的对象都被标记以后,主线程会和几个子线程一起,对一些内存page进行压缩和更新指针的工作,不是所有的page都需要进行压缩(只对高碎片化的进行压缩),不需要压缩的利用free-list进行打扫。

![Major GC同步调度](http://img.pandihai.com/09.svg)<center><font color=gray size=2>Major GC同步调度</font></center>


### 什么时候会执行GC
在JavaScript中我们没办法用编程的方式主动触发GC,因为涉及到复杂的线程调度,主动的触发GC可能会影响正在执行的GC或者下次的GC。对于Scavenger来说,当在新生代中分配内存时,已经没有空间分配内存导致分配内存失败时,开始Scavenger垃圾回收,希望能释放一些内存,然后在尝试重新分配内存。对于老代来说,开启内存回收的时机要复杂很多,简单来说会根据老代中内存占用的百分比和需要被分配对象的百分比计算出一个合适的阀值,触及到这个阀值就会开启老代的垃圾回收。

我们可以通过手动设置来设置新生代和老代的space大小:
```js
    node --max-old-space-size=1700 index.js
    node --max-new-space-size=1024 index.js
```


#### 空闲时GC
虽然我们通过JavaScript没办法主动触发GC,但是在V8中还有一个空闲GC的机制,他根据被嵌入宿主来决定什么时候属于空闲时来执行GC。比如V8在Chrome浏览器中,为了保证动画渲染的流畅,一秒钟需要渲染60个帧,相当于16.6毫秒渲染一帧,在16.6毫秒以内渲染完了一帧,比如只花了10毫秒就渲染完了这一帧的动画,那么你就有了6.6毫秒的空闲时间可以执行一些空闲时的GC(在许多新版本的浏览器中,开发者也可以通过[requestIdleCallback](https://developers.google.com/web/updates/2015/08/using-requestidlecallback)事件,利用浏览器空闲时间来提高性能,有兴趣的可以去了解[React 16 fiber的实现](https://www.youtube.com/watch?v=ZCuYPiUIONs))。

![空闲时GC](http://img.pandihai.com/10.svg)<center><font color=gray size=2>利用主线程空闲时间进行GC</font></center>

#### Incremental
那么在空闲的几毫秒时间里能完成一次GC吗?那就是接下来就要介绍另外一种调度算法Incremental了,相比较于其他调度算法在暂停一次主线程执行一整次完成的GC,Incremental要求把一整个GC中的工作拆成一小块,和主线程中的JS递进的执行,或者在主线程有空闲时间的时候执行一小块GC任务。

![Incremental](http://img.pandihai.com/06.svg)<center><font color=gray size=2>将一整个GC切分成一小块GC任务,插入到主线程中进行</font></center>

### 总结
不同JavaScript引擎实现GC都有不同程度的差异,本文主要以V8为例,有很多地方没有非常仔细的展开,比如：其实老代里面不是只有一块space,而是有4块space组成,每块space存放着不同的数据(old space,large object space,matedata space,code space)。垃圾回收设计本身就是一个很复杂的程序,有了GC,让开发者可以完全不用担心内存的管理问题。但是适当的了解垃圾回收的原理能够帮助我们更加深入的理解JavaScript的运行环境,也可以帮助我们写出更高效率的代码。

最后的最后将之前的内存泄漏代码一步步的推演:

1. 首先在全局作用域中声明了两个变量theThing和replaceThing,其中replaceThing被赋值为一个方法(callable object),然后调用setInterval方法,每隔1000毫秒调用一次replaceThing。
2. 1000毫秒到了,执行replaceThing,创建一个新的局部作用域,根据hoist,先将方法unused方法声明,然后声明了originThing和c变量。这里特别要注意,**闭包是在方法声明的时候被创建的而不是在方法执行的时候创建的**,所以当声明了unused方法以后,同时创建了一个闭包,里面包含了unused方法使用的局部作用域变量originThing。**另外在V8中一旦作用域有闭包,这个上下文会被绑定到所有方法当中作为闭包,即使这个方法没有使用这个作用域中的任何一个变量**,所以在这里给全局作用域赋值的时候,someMethod作为一个方法,也被绑定一个unused创建的闭包,且被赋值在全局作用域中的theThing上了。
3. 如果这时候开始第一次GC,从全局对象进行Reachable分析:theThing(reachable),replaceThing(reachable),theThing->longStr(reachable),theThing->someMethod(reachable),execution stack -> setInterval -> closure -> originThing(reachable)。   
所有标记完成。此时:
```js
          from-space                                to-space

    theThing         (reachable)                theThing
    replaceThing     (reachable)                replaceThing
    unused                                      originThing
    originThing      (reachable)       =>       longStr  
    c                                           someMethod
    longStr          (reachable)                
    someMethod       (reachable)                
```
4. 在过1000毫秒以后又执行replaceThing,又执行一遍步骤2
5. 第二次GC开始

```js
          from-space                                to-space                           old-space

    theThing         (reachable)                theThing                             originThing -> theThing
    replaceThing     (reachable)                replaceThing                         theThing -> longStr
    unused                                      originThing                          theThing -> someMethod
    originThing      (reachable)       =>       longStr                    =>        someMethod -> originThing(closure)        
    c                                           someMethod
    longStr          (reachable)                
    someMethod       (reachable)                
```
6. 因为闭包一直连着这originThing,导致了old-space中的originThing一直无法释放。随着时间的推移,每个1000毫秒执行一次replaceThing方法

```js
         old-space
    originThing -> theThing -> longStr & someMethod -> originThing(closure)
    originThing -> theThing -> longStr & someMethod -> originThing(closure)
    originThing -> theThing -> longStr & someMethod -> originThing(closure)
    originThing -> theThing -> longStr & someMethod -> originThing(closure)
    originThing -> theThing -> longStr & someMethod -> originThing(closure)
```

#### 结论 
主要导致内存泄漏的原因是

![闭包是在声明的时候被创建的](http://img.pandihai.com/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20190708095349.png)<center><font color=gray size=2>闭包是在声明的时候被创建的,而不是执行的时候被创建的。</font></center>

然后导致在originalThing还引用着老的theThing,theThing中的someMethod引用着originalThing导致全部都reachable无法释放。

``` javacript

var theThing = null;
var replaceThing = function () {
  var originalThing = theThing;
  var c = 'a'
  function unused() {
    if (originalThing) {
      console.log("hi");
    }
  };

  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log('1111');
    }
  };

  originalThing = null;    //手动释放局部作用域中的变量
};

setInterval(replaceThing,1000)

```
