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
  var originalThing = new Array(1000000).join('*');
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

## 简单了解 Tracing/Reachable 追踪分析
GC的第一步就是要找出哪些对象需要被回收,哪些不需要。在追踪分析(Tracing/Reachable)中,认为可以被追踪到(reachability)的对象认为是不能被回收的对象,剩下的不能被追踪到的对象就是要回收的对象。
在V8中,追踪分析会从根对象开始(GC Root)根据指针将所有被能被追踪到的对象标记为reachable,javascript中根对象包括调用堆栈和global对象。


## The Generational Hypothesis
Generational Hypothesis的意思是大部分的对象在早期就需要被回收。基于这样的一个假设,有很多的编程语言的垃圾回收机制在设计的时候都是将内存分代,年轻代(young generation),和老代(old generation)。
这里的代其实就是开辟两块space分别存储刚被分配的对象和经过两次GC还是没有被回收的对象。在V8中有两个垃圾回收器分别对年轻代和老代进行垃圾回收,Scavenger针对年轻代进行垃圾回收,Major GC针对老代进行垃圾回收,他们的算法也是不同的。

### Scavenger
V8在年轻代的内存space使用的是semi-space算法,也就是说将内存分为两半,同时只有一块的内存能被使用,另外一半是完全空的(或者说这一半内存都是可以被分配的)。在程序开始执行的时候,将所有的变量都分配可以被使用的一半内存中(叫做from-space)。当第一次GC开始的时候根据追踪分析结果,将所有可以reachable的对象(不能被释放的对象),全部转移到剩余一半可以被分配的内存中(to-space)，这样from-space中的内存又全部可以被分配了，这个时候如果又有新申明的对象需要分配内存,就会分配到这一块内存当中了,最后在转移完不能被释放的对象之后,还需要更新引用指针,指向在to-space中最新的地址。

![第一次GC](http://img.pandihai.com/03.svg)<center><font color=gray size=2>第一次GC</font></center>


第二次GC开始的时候,在原本的to-space中仍然不能被释放的对象首先转移到老代(old generation)的space中,这时候to-space中又全部可以被分配,重复之前的操作。从from-space中将不能被释放的对象转移过来。完成2次GC之后,存货了两次的对象现在就在老代里面了,而存活一次GC的对象现在就在to-space中了,这个to-space也被叫做intermediate generation(中生代).在Scavenger中回收内存有三个过程:标记(追踪分析),转移(from-space to to-space),更新指针地址。他们是交错进行的,不是在同一个进程内依次进行的,后面会讲到。

![第二次GC](http://img.pandihai.com/04.svg)<center><font color=gray size=2>第二次GC</font></center>


在这种内存回收的机制中,其中一个问题就是转移对象的时候是会消耗一定性能的,但是根据Generational Hypothesis的假设大部分的对象在早期就会被回收了,这也就意味着只有少部分不能被回收的对象需要被移动，这也意味着如果这个假设不成立，比如我们的代码中有很多的闭包导致很多的作用域不能被释放,那么将会有大量的对象需要在space之间转移,是比较浪费性能的。但是相反的,基于大部分对象都可以在早期被回收的假设,如果大部分的对象在早期就可以被释放,这种机制的内存回收对这需要在早期就回收的对象其实是什么都不需要做的,只需要把不能释放的少部分对象进行转移（from-space to to-space）,然后在下次分配内存的时候把这部分需要释放的对象所占的内存直接覆盖就可以了(rewrite dead object)。

### Major GC
Major GC主要负责老代的内存回收,同样也是三个过程:标记(追踪分析),清除,整理压缩内存。标记这一步和Scavenger一样通过追踪分析确定哪些内存需要被回收,然后在对象被回收以后将被回收的内存加入到free-list这个数据结构中,free-list就像是一个个抽屉,每个抽屉的大小代表了从这个地址开始可以被连续分配的内存的大小,当我们需要在老代中重新分配内存的时候就可以快速的根据需要分配内存的大小找到一个合适的抽屉把内存进行分配。最后就是进行内存整理,这个就好像是Windows系统整理磁盘一样,将还没被幸存的对象利用free-list查找拷贝到其他的已经被整理完的page中,这样使小块的内存碎片也被整理完之后加以利用。跟Scavenger中一样来回拷贝对象也会有性能的消耗,在V8中只会对高度碎片化的page进行整理,对其他的page进行清除,这样在转移的时候也是一样的只需要转移存活的对象就可以了。