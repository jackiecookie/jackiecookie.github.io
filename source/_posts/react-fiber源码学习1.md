---
layout: post
title: react-fiber源码学习1
date: 2018-01-12 14:52:19
tags:
---

##准备

react-render可以允许我们自己定义的render把react组件渲染到不同的环境中,现在已经可以渲染在很多的平台中比如:浏览器(react-dom),ios装置(react-native),安卓(react-native),VR(react-vr),pdf(react-pdf)等等地方,完成一个对应环境的config就可以把组件渲染到对应的环境中.而使用的react的底层核心代码(Fiber,Scheduler,Reconciler)是同一个,这里就从自己比较熟悉的浏览器前端来介绍源码.

### 什么是react-fiber?
fiber是react6.0以后引入的一个新的核心算法,react历时了两年重新重写了react的核心算法.

### fiber能带来什么?
我们都知道javascript是一个单线程的语言,以浏览器为例之前的react在浏览器中使用调用堆栈来管理操作,而react引入了一个优先级的新概念(优先级之前在ng中已经被提出过),也重新自己实现了stack,我们可把一个fiber理解为一个stack帧(stack frame)他把每一次操作和渲染做切分为3个阶段你可以暂停,停止,重新启动一个fiber,也可以将优先级更高的操作优先执行,从而来带来更流畅的动画和效率.听起来很hack,他到底是怎么做到的呢?我们试着从源码里面找一下答案.

### reconciler
为了解释fiber就需要提到reconciler,前面已经提到过react允许我们把组件渲染到各种平台和环境中,react可以做到这个的原因就是reconciler,而fiber也是reconciler的一部分,fiber重新定义了reconciler,以浏览器为例:reconciler计算了dom树哪里发生了改变,render在根据发生的改变把他们渲染到不同的平台上面去.

### scheduler
前面提到react引入了优先级的概念,scheduler就是根据优先级来调配什么任务需要优先被执行.比如没有在页面中显示的元素他的处理优先级就不应该在一个动画的前面.而在原先的回调堆栈中,回调堆栈只能根据放入堆栈中的执行顺序来依次执行,直到执行到堆栈中没有方法为止.


## 初探
前面已经提到过fiber可以被认为是stack中的stackFrame,一个组件不论是host组件还是自定义组件同时最多存在两个fiber,正在执行中(workInProgress fiber)的fiber和已经输出的fiber(current fiber),后面解释为什么需要两个fiber,先来看一下fiber的字段.
``` JavaScript
Fiber = {|
 
  // fiber的类型 包括有ClassComponent,HostRoot,HostComponent等等
  tag: TypeOfWork,

  // 识别fiber的唯一值
  key: null | string,

  // 根据类型分配的值 可能是用来实例化组件的方法等等
  type: any,

  // 这个值通常是指向的父级元素,就像Stack Frame返回地址一样
  return: Fiber | null,

  // 下面的值是fiber的子fiber或者兄弟fiber用来遍历subtree来开启任务或者提交任务
  child: Fiber | null,
  sibling: Fiber | null,
  index: number,
  
  // 组件的属性值
  pendingProps: any, 
  memoizedProps: any, 
  
  // 更新操作的队列
  updateQueue: UpdateQueue | null,
  
  // 组件的状态
  memoizedState: any,
  
  // 变化类型
  effectTag: TypeOfSideEffect,

  // 下一个受影响的fiber用于提交更新是使用
  nextEffect: Fiber | null,
  firstEffect: Fiber | null,
  lastEffect: Fiber | null,

  // fiber的优先级
  pendingWorkPriority: PriorityLevel,

  //另外一个fiber
  alternate: Fiber | null,

|};
```
以下面的组件为例来看看fiber是什么样子的
``` JavaScript
class ExampleApplication extends React.Component {
      constructor() {
         super();
       }
       render() {
         return (
           <div>
             <h3>
              Fiber例子
             </h3>
             <div>
             </div>
           </div>
         );
       }
     }
     ReactDOM.render(
             <ExampleApplication size="0" />,
         document.getElementById('container')
     );
```
他生成的结构是这样的
![react-fiber](http://image.jiantuku.com/18-1-15/11117894.jpg?e=1515998410&token=el7kgPgYzpJoB23jrChWJ2gV3HpRl0VCzFn8rKKv:spwyqbyT3s73UGF_ielfxZbfq4E=)
而因为是渲染所以他们的优先级都是SynchronousPriority.

## 深入
当react开始执行上面这个渲染的时候,会先生成当前container的root-fiber将他传入`scheduleUpdate`中在这个方法中,在这个方法中主要执行下面几件事:
1.如果是有正在执行的任务,并且当前任务优先级高于正在执行的任务,那么将下一个任务清空.
``` JavaScript
if (!isPerformingWork && priorityLevel <= nextPriorityLevel) {
      nextUnitOfWork = null;
    }
```
2.将root-fiber放入Scheduled队列中.
``` JavaScript
scheduleRoot(root, priorityLevel);
```
3.根据优先级执行`performWork`方法,如果是比较低优先级的任务那么就安排一个延迟任务,在高级浏览器支持的情况下这里使用了[requestIdleCallback](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)方法他会在浏览器空闲的时候调用函数,不支持的情况也有polyfill.
``` JavaScript
switch (priorityLevel) {
 case SynchronousPriority:
    if (isUnbatchingUpdates) {
      performWork(SynchronousPriority, null);
    } else {
      performWork(TaskPriority, null);
    }
    break;
  default:
    if (!isCallbackScheduled) {
      scheduleDeferredCallback(performDeferredWork);
      isCallbackScheduled = true;
    }
}
```
这里的`priorityLevel`字段是当前这次操作的优先级等级,这次操作可能是一个setState操作也可能是依次render操作,或是是由`unstable_deferredUpdates`方法包裹的一个延迟操作方法,像这样:
``` JavaScript
ReactDOM.unstable_deferredUpdates(() => {
              this.setState();
            });
```
 那么他就会设置一个priorityContext优先级设置为LowPriority的依次操作.在后面通过`getPriorityContext`方法获取这次操作优先级的使用就会被指定为优先级.
``` JavaScript
function getPriorityContext(
  fiber: Fiber,
  forceAsync: boolean,
): PriorityLevel {
  let priorityLevel = priorityContext;
  if (priorityLevel === NoWork) {
    if (
      !useSyncScheduling ||
      fiber.internalContextTag & AsyncUpdates ||
      forceAsync
    ) {
      priorityLevel = LowPriority;
    } else {
      priorityLevel = SynchronousPriority;
    }
  }
  if (
    priorityLevel === SynchronousPriority &&
    (isPerformingWork || isBatchingUpdates)
  ) {
    return TaskPriority;
  }
  return priorityLevel;
}
```
我们知道了比较关键的priorityLevel字段是哪里来的以后就要看接下来要执行的`performWork`方法,
在这个方法中除了大部分性能监控错误捕捉的代码外(react中有很多while循环的代码,那么在性能监控,调试和错误捕捉上做了大量的工作,有机会的话看看能不能在后面研究一下),
1.主要执行了`workLoop`这个比较重要的方法,而这个方法我认为是类似与回调stack的实现.
2.如果`workLoop`执行完之后还有剩余的任务需要执行那么就再执行`requestIdleCallback`方法.为什么会有剩余的任务没执行完呢?在看到`workLoop`方法的时候我们就知道了.
``` JavaScript
if (nextPriorityLevel > TaskPriority$1 && !isCallbackScheduled) {
      scheduleDeferredCallback(performDeferredWork);
      isCallbackScheduled = true;
}
```
在看`workLoop(minPriorityLevel, deadline)`之前我们先看一下他的第二个deadline参数,在浏览器空闲执行`requestIdleCallback`方法的时候,同时会传入一个deadline的参数,
通过这个参数调用timeRemaining方法可以获得剩余的空闲时间,那么我们就可以在workLoop执行优先级没那么高的任务时,根据空闲时间来暂停或者继续执行任务了.
那接着我们就着重来看一下这个`workLoop`方法,先看这行代码
``` JavaScript
if (nextUnitOfWork === null) {
      resetNextUnitOfWork();
}
```
1.而在`resetNextUnitOfWork`方法中做了这几件事:
    1.1 清除Scheduled队列中优先级为NoWork的任务.
``` JavaScript
while (
      nextScheduledRoot !== null &&
      nextScheduledRoot.current.pendingWorkPriority === NoWork
    ) {
      nextScheduledRoot.isScheduled = false;
      const next = nextScheduledRoot.nextScheduledRoot;
      nextScheduledRoot.nextScheduledRoot = null;
      if (nextScheduledRoot === lastScheduledRoot) {
        nextScheduledRoot = null;
        lastScheduledRoot = null;
        nextPriorityLevel = NoWork;
        return null;
      }
      nextScheduledRoot = next;
    }
```
1.2 找到优先级别最高的任务,设置为下一个要开始的任务.
    
``` JavaScript
 while (root !== null) {
      if (
        root.current.pendingWorkPriority !== NoWork &&
        (highestPriorityLevel === NoWork ||
          highestPriorityLevel > root.current.pendingWorkPriority)
      )  {
        highestPriorityLevel = root.current.pendingWorkPriority;
        highestPriorityRoot = root;
      }
      root = root.nextScheduledRoot;
    }
    if (highestPriorityRoot !== null) {
      nextPriorityLevel = highestPriorityLevel;
      resetContextStack();

      nextUnitOfWork = createWorkInProgress(
        highestPriorityRoot.current,
        highestPriorityLevel,
      );
      if (highestPriorityRoot !== nextRenderedTree) {
        nestedUpdateCount = 0;
        nextRenderedTree = highestPriorityRoot;
      }
      return;
    }
```
如果`nextUnitOfWork === null`,我们之前在`scheduleUpdate`所做的事的第一点中提到过如果有正在执行的任务,并且当前任务优先级高于正在执行的任务nextUnitOfWork会被置空,那么到了这里正在执行的低优先级的任务就被暂停了,然后重新找到优先级最高的任务重新作为下一个任务.也就是fiber被暂停,并开始优先级高的任务.
找到优先级最高的任务之后就开始循环执行了
``` JavaScript
loop: do {
      if (nextPriorityLevel <= TaskPriority) {
        while (nextUnitOfWork !== null) {
          nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
          if (nextUnitOfWork === null) {
            priorityContext = TaskPriority;
            commitAllWork(pendingCommit);
            priorityContext = nextPriorityLevel;
            handleCommitPhaseErrors();
            if (
              nextPriorityLevel === NoWork ||
              nextPriorityLevel > minPriorityLevel ||
              nextPriorityLevel > TaskPriority
            ) {
              break;
            }
          }
        }
      } else if (deadline !== null) {
        while (nextUnitOfWork !== null && !deadlineHasExpired) {
          if (deadline.timeRemaining() > timeHeuristicForUnitOfWork) {
            nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
            if (nextUnitOfWork === null) {
              if (deadline.timeRemaining() > timeHeuristicForUnitOfWork) {
                priorityContext = TaskPriority;
                commitAllWork(pendingCommit);
                priorityContext = nextPriorityLevel;
                handleCommitPhaseErrors();
                if (
                  nextPriorityLevel === NoWork ||
                  nextPriorityLevel > minPriorityLevel ||
                  nextPriorityLevel < HighPriority
                ) {
                  break;
                }
              } else {
                deadlineHasExpired = true;
              }
            }
          } else {
            deadlineHasExpired = true;
          }
        }
      }
      switch (nextPriorityLevel) {
        case SynchronousPriority:
        case TaskPriority:
          if (nextPriorityLevel <= minPriorityLevel) {
            continue loop;
          }
          break loop;
        case HighPriority:
        case LowPriority:
        case OffscreenPriority:
          if (deadline === null) {
            break loop;
          }
          if (!deadlineHasExpired && nextPriorityLevel <= minPriorityLevel) {
            continue loop;
          }
          break loop;
        case NoWork:
          break loop;
      }
    } while (true);
```
这个循环中主要分为两种情况:
1.优先级为TaskPriority和SynchronousPriority的任务,也就是马上要执行的任务,那他就会立马调用`performUnitOfWork`执行.
2.剩下就作为延迟任务,前面已经提到过`deadline.timeRemaining()`可以获得还剩多少空闲时间来执行任务,那么在空闲时间足够的情况下就可以执行`performUnitOfWork`方法了.
在`performUnitOfWork`方法中主要是执行两个方法:
1.beginWork开始一个fiber任务,给当前执行的fiber的component进行实例化调用实例的render方法,调用`shouldComponentUpdate`比较前后state和prop差异,然后生成然后返回他的child fiber,如果没有child fiber则返回一个null.
2.在没有child fiber的情况下执行`completeUnitOfWork`,在`completeUnitOfWork`中执行`prepareUpdate`比较fiber的diff,如果存在diff加入到update队列中,标记影响顺序.然后在有兄弟节点的情况下返回兄弟fiber,在没有兄弟fiber的情况下一直向上向父级继续执行completeUnitOfWork.
最后在fiber为root fiber的时候将pendingCommit赋值为root fiber在下一次任务开始的时候就会进入到`commitAllWork`阶段了,在`commitAllWork`方法中就直接提交差异将子fiber节点加入到父fiber节点中了.

到这里可以看到 一个fiber更新 将经历下面几个阶段
1.beginWork:           创建实例,比较state对象差异.
2.completeUnitOfWork:  调用`prepareUpdate`比较fiber diff,标记side affect,加入update队列.
3.commitAllWork:       提交差异更新节点.
其中对于延迟任务来说,在第1和第2个阶段可以被打断,在第三个阶段会一次性提交所有差异,且无法打断.
相反对于高优先级别的任务比较简单,会连贯的完成三个阶段.
一个fiber树在渲染的时候三个阶段的顺序如图
![react-fiber阶段执行顺序](http://image.jiantuku.com/18-1-16/51726724.jpg?e=1516086010&token=el7kgPgYzpJoB23jrChWJ2gV3HpRl0VCzFn8rKKv:tQWcGjlBexVCSyT_tG7FS99PmRU=)
前面提到一个组件最多存在两个fiber,fiber的alternate字段指向各自的另外一个fiber,也就是说current fiber的alternate是work-In-Progress Fiber,而work-In-Progress Fiber的alternate指向current fiber.一个fiber在经历三个阶段的时候,在最后的一个阶段才会提交改动,workInProgress fiber的存在是不希望在前两个阶段计算差异的时候去改变dom,而能够在最后一个阶段一次性提交改动,react在这里使用了双缓存池技术,因为在一个组件最多由两个fiber的情况下,缓存一个fiber可以重用和避免分配内存是带来的开销.

最后用[https://github.com/claudiopro/react-fiber-vs-stack-demo](https://github.com/claudiopro/react-fiber-vs-stack-demo)来梳理一下fiber是如何schedule任务的.
首先看到这个demo中的这段代码.
``` JavaScript
var start = new Date().getTime();
function update() {
  ReactDOMFiber.render(
    <ExampleApplication elapsed={new Date().getTime() - start} />,    document.getElementById('container')
  );
  requestAnimationFrame(update);
}
requestAnimationFrame(update);
```

他定义了一个递归任务,在每一帧里更新ExampleApplication的elapsed属性,让图形最后得到一个动画的效果.这里的每次`render`方法都是一个最高级别SynchronousPriority级别优先级的任务.
接着看到tick方法中定义了一个延迟操作,他会在每一秒钟更新seconds的值,因为他是一个延迟的操作,也就是说这个setState操作的优先级是LowPriority的.
``` JavaScript
 componentDidMount() {
          this.intervalID = setInterval(this.tick, 1000);
        }
        
tick() {
          ReactDOMFiber.unstable_deferredUpdates(() =>
            this.setState(state => ({ seconds: (state.seconds % 10) + 1 }))
          );
        }
```
在Stack的版本中所有的操作都依次的加入到回调stack中被依次调用,画面出现了卡顿,托帧的效果.
相反的在fiber的版本的因为有了优先级的存在.低优先级的任务会在浏览器渲染空闲回调时间允许的时间里执行,没有出现卡顿,得到一个流畅的效果,虽然seconds没有被及时的更新但是我们认为一个低优先的任务被延迟是没关系的.让我们想象一下:
现在`render`正在更新`elapsed`属性,fiber树立刻开始执行在没有延迟的情况下执行完了三个阶段,并且提交了渲染结果.在requestAnimationFrame的不停调用下这个方法也在不停的被执行.
这是延迟操作setState被执行了,因为是一个LowPriority的操作,这个操作将在浏览器空闲的时候被调用.
这个时候浏览器空闲了,开始执行第一个fiber的beginWork操作返回了一个child作为下一个任务.正在准备执行下一个任务的时候发现空闲时间已经不足了也就是说`deadline.timeRemaining() > timeHeuristicForUnitOfWork = false;`.浏览器又接受到了新的render方法的更新值.
这时`scheduleUpdate`中的`if (!isPerformingWork && priorityLevel <= nextPriorityLevel)`成立`nextUnitOfWork = null;`下次`resetNextUnitOfWork`又重新找到了现在优先级最高的`render`操作开始执行`render`.
等浏览器又有空闲的时候之前没有结束的`setState`操作又被执行,**之前执行过的fiber他的priority已经为NoWork然后被跳过**,直接达到没有上次被暂停的任务开始继续执行,就这样好几个空闲时间之后一次`setState`操作终于走完前面两个阶段获得了diff终于可以在空闲的时间执行了commit操作提交了diff.至此完成的`setState`完成也没有影响`render`动画.


react fiber的初探暂时就到这里,关于这个版本的react有很多新的东西可以展开,比如错误绑定,性能监控,开发调试.另外也可以定义自己的Custom Renderer.可以学到很多东西,也希望有兴趣的朋友一起交流指出,谢谢!