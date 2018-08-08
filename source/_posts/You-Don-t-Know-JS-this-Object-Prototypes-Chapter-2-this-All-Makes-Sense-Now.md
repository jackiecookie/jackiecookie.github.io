---
layout: post
title: 'You Don''t Know JS: this & Object Prototypes-Chapter 2: this All Makes Sense Now'
date: 2017-07-29 12:19:26
tags:
---

# You Don't Know JS: *this* & Object Prototypes
# Chapter 2: `this` All Makes Sense Now!

# 你不知道的JS:*this* & 对象原型
# 第二章:现在`this`的一切都有了意义

In Chapter 1, we discarded various misconceptions about `this` and learned instead that `this` is a binding made for each function invocation, based entirely on its **call-site** (how the function is called).

在第一章中，我们抛弃了各种有关`this`的错误的观点，然后学习了`this`其实是一个为每个方法执行进行的一个绑定，完全以**调用位置**为基础(方法如何被调用)。

## Call-site

## 调用位置

To understand `this` binding, we have to understand the call-site: the location in code where a function is called (**not where it's declared**). We must inspect the call-site to answer the question: what's *this* `this` a reference to?

为了理解`this`绑定，我们需要了解调用位置:在代码中一个方法被调用的位置(**不是他在哪里声明**)。我们必须检查调用位置来回答这个问题:这个`this`引用着什么?

Finding the call-site is generally: "go locate where a function is called from", but it's not always that easy, as certain coding patterns can obscure the *true* call-site.

通常找到调用位置是:"到一个方法从哪里被调用的位置"，但是这不是每次都那么简单，例如某一种代码模式会混淆*真正*的调用位置。

What's important is to think about the **call-stack** (the stack of functions that have been called to get us to the current moment in execution). The call-site we care about is *in* the invocation *before* the currently executing function.

重要的是要考虑**调用堆栈**(方法调用的堆栈使我们能到达当前执行的瞬间)。调用位置我们关心的是在我们当前执行方法之前的调用。

Let's demonstrate call-stack and call-site:

让我们来演示调用堆栈和调用位置:

```js
function baz() {
    // call-stack is: `baz`
    // so, our call-site is in the global scope

    console.log( "baz" );
    bar(); // <-- call-site for `bar`
}

function bar() {
    // call-stack is: `baz` -> `bar`
    // so, our call-site is in `baz`

    console.log( "bar" );
    foo(); // <-- call-site for `foo`
}

function foo() {
    // call-stack is: `baz` -> `bar` -> `foo`
    // so, our call-site is in `bar`

    console.log( "foo" );
}

baz(); // <-- call-site for `baz`
```

Take care when analyzing code to find the actual call-site (from the call-stack), because it's the only thing that matters for `this` binding.

当分析代码来寻找真正的调用位置时(从调用堆栈里)需要小心，因为这个是`this`绑定唯一在意的事情。

**Note:** You can visualize a call-stack in your mind by looking at the chain of function calls in order, as we did with the comments in the above snippet. But this is painstaking and error-prone. Another way of seeing the call-stack is using a debugger tool in your browser. Most modern desktop browsers have built-in developer tools, which includes a JS debugger. In the above snippet, you could have set a breakpoint in the tools for the first line of the `foo()` function, or simply inserted the `debugger;` statement on that first line. When you run the page, the debugger will pause at this location, and will show you a list of the functions that have been called to get to that line, which will be your call stack. So, if you're trying to diagnose `this` binding, use the developer tools to get the call-stack, then find the second item from the top, and that will show you the real call-site.

**注意:** 你可以在你的脑海里通过按照顺序查看方法调用调用链来呈现一个调用堆栈，就像我们在代码片段中做的注释一样。但是这个是需要细心的而且易于出错。看调用堆栈的另外一个方式是使用你浏览器的调试工具。大多数现代桌面浏览器都有内置的开发者工具包含一个JS调试器。在上面的代码片段中，你可以在工具中在`foo()`方法的第一行设置一个断点，或者简单的在第一行插入一个`debugger;`表达式。当你运行到这一页，调试器会在这个位置暂停，然后将会展示一个在那一行已经被调用的方法列表，这就是你的调用堆栈。所以，如果你试图判断`this`绑定，使用开发者工具获得调用堆栈，然后从最上面找到第二项，他将会向你展示真正的调用位置。

## Nothing But Rules

## 只有规则

We turn our attention now to *how* the call-site determines where `this` will point during the execution of a function.

现在将我们的注意力转到调用位置是*如何*确定`this`在一个执行的方法中会指向什么的。

You must inspect the call-site and determine which of 4 rules applies. We will first explain each of these 4 rules independently, and then we will illustrate their order of precedence, if multiple rules *could* apply to the call-site.

你必须检查调用位置然后决定应用4个规则中的哪一个。首先我们将会各自解释4个规则中每一个，然后我们将会举例说明如果多个规则可以应用于一个调用位置他们的优先顺序。

### Default Binding

### 默认绑定

The first rule we will examine comes from the most common case of function calls: standalone function invocation. Think of *this* `this` rule as the default catch-all rule when none of the other rules apply.

我们将要探索的第一个规则是从最常见的方法调用情况中来的:独立方法调用。把这个`this`规则当做是当没有其他规则应用时的默认规则它是捕获全部的。

Consider this code:

考虑这个代码：

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo(); // 2
```

The first thing to note, if you were not already aware, is that variables declared in the global scope, as `var a = 2` is, are synonymous with global-object properties of the same name. They're not copies of each other, they *are* each other. Think of it as two sides of the same coin.

如果你还没有意识到,需要注意的第一件事，`var a = 2`中变量被声明在全局作用域中，他们是全局对象相同名字的属性的同义词。他们不是相互拷贝的，他们就是对方。想象一下一个硬币的两面。

Secondly, we see that when `foo()` is called, `this.a` resolves to our global variable `a`. Why? Because in this case, the *default binding* for `this` applies to the function call, and so points `this` at the global object.

其次，我们看到当`foo()`被调用，`this.a`指向的是我们的全局变量`a`(resolves:to reach a decision by means of a formal vote:根据投票来达成一个决定)。为什么？因为在这个例子中，在这个方法调用中`this`绑定规则应用了*默认绑定*，所以`this`指向了全局作用域。

How do we know that the *default binding* rule applies here? We examine the call-site to see how `foo()` is called. In our snippet, `foo()` is called with a plain, un-decorated function reference. None of the other rules we will demonstrate will apply here, so the *default binding* applies instead.

我们怎么知道*默认绑定*规则在这里应用了?我们探究一下调用位置来看`foo()`是如何被调用的。在我们的代码片段中，`foo()`是一个平淡无奇的调用，未经装饰的方法引用。没有任何其他我们将展示的规则会在这里应用，所以*默认绑定*在这里被应用了。

If `strict mode` is in effect, the global object is not eligible for the *default binding*, so the `this` is instead set to `undefined`.

如果受了`strict mode`(严格模式)的影响，全局对象作为 *default binding* 将被认为不合法，所以`this`会被设置为`undefined`。

```js
function foo() {
	"use strict";

	console.log( this.a );
}

var a = 2;

foo(); // TypeError: `this` is `undefined`
```

A subtle but important detail is: even though the overall `this` binding rules are entirely based on the call-site, the global object is **only** eligible for the *default binding* if the **contents** of `foo()` are **not** running in `strict mode`; the `strict mode` state of the call-site of `foo()` is irrelevant.

一个微妙但是重要的细节是:即使所有的`this`绑定规则都是完全基于调用位置的，如果`foo()`的**内容不是**运行在`strict mode`(严格模式)下那么全局对象是**唯一**合法的*默认绑定*；`foo()`的调用位置和`strict mode`(严格模式)的状态不相关。(即不受影响)

```js
function foo() {
	console.log( this.a );
}

var a = 2;

(function(){
	"use strict";

	foo(); // 2
})();
```

**Note:** Intentionally mixing `strict mode` and non-`strict mode` together in your own code is generally frowned upon. Your entire program should probably either be **Strict** or **non-Strict**. However, sometimes you include a third-party library that has different **Strict**'ness than your own code, so care must be taken over these subtle compatibility details.

**注意:** 有意的将`strict mode`和非`strict mode`在你的代码中混合在一起使用通常是不被推荐的。你的整个程序需要最好只使用**严格模式**或者**非严格模式**其中之一。然而，有时候你会在你的代码中使用和你**严格模式**不一样的第三方库，所以必须小心处理这些微妙的兼容细节。

### Implicit Binding

### 隐含绑定

Another rule to consider is: does the call-site have a context object, also referred to as an owning or containing object, though *these* alternate terms could be slightly misleading.

另外一个需要考虑的规则是:调用位置是否有一个上下文对象，也成为拥有或者容器对象，尽管*这些*交替出现的术语可能会有些误导。

Consider:

考虑一下:

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

obj.foo(); // 2
```

Firstly, notice the manner in which `foo()` is declared and then later added as a reference property onto `obj`. Regardless of whether `foo()` is initially declared *on* `obj`, or is added as a reference later (as this snippet shows), in neither case is the **function** really "owned" or "contained" by the `obj` object.

首先，注意到`foo()`声明的行为然后接下里作为一个引用属性被附加到`obj`上。不管`foo()`是初始化声明在`obj`上,或者作为一个引用加到上面(就像这段代码一样),这两种情况都是**方法**真正的被`obj`对象"拥有"或者"包含"。

However, the call-site *uses* the `obj` context to **reference** the function, so you *could* say that the `obj` object "owns" or "contains" the **function reference** at the time the function is called.

然而，调用位置*使用*`obj`上下文来**引用**方法，所以你*可以*说`obj`对象在方法调用的时候"拥有"或者"包含"**方法引用**。

Whatever you choose to call this pattern, at the point that `foo()` is called, it's preceded by an object reference to `obj`. When there is a context object for a function reference, the *implicit binding* rule says that it's *that* object which should be used for the function call's `this` binding.

不论你选择如何称呼这种模式，`foo()`的被调用是通过一个`obj`对象引用作为引导的。当一个方法引用使用一个上下文对象时，*隐晦绑定* 规定规定*那个*对象需要被用作为方法执行的`this`绑定值。

Because `obj` is the `this` for the `foo()` call, `this.a` is synonymous with `obj.a`.

因为`obj`是 `foo()`调用的`this`，所以`this.a`是`obj.a`的同义词。

Only the top/last level of an object property reference chain matters to the call-site. For instance:

调用位置只跟对象属性引用链的最高级/最后级相关。举例来说：

```js
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```

#### Implicitly Lost

#### 隐晦丢失

One of the most common frustrations that `this` binding creates is when an *implicitly bound* function loses that binding, which usually means it falls back to the *default binding*, of either the global object or `undefined`, depending on `strict mode`.

`this`绑定的创建最常见的挫败是当一个*隐晦绑定*方法丢失了那个绑定，这通常意味着他退回到*默认绑定*，`this`会是全局对象或者`undefined`二者之一，取决于是否`strict mode`(严格模式)。

Consider:

考虑一下:

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // function reference/alias!

var a = "oops, global"; // `a` also property on global object

bar(); // "oops, global"
```

Even though `bar` appears to be a reference to `obj.foo`, in fact, it's really just another reference to `foo` itself. Moreover, the call-site is what matters, and the call-site is `bar()`, which is a plain, un-decorated call and thus the *default binding* applies.

尽管`bar`似乎是一个指向`obj.foo`的引用，事实上，他只是`foo`本身的另外一个引用。此外，调用位置和什么相关，调用位置是`bar()`，他是一个平淡无奇的调用，未经装饰的调用因此*默认绑定*应用了。

The more subtle, more common, and more unexpected way this occurs is when we consider passing a callback function:

更微妙，更常见，更意料之外的这种情况的发生是当我们考虑传递一个回调方法时:

```js
function foo() {
	console.log( this.a );
}

function doFoo(fn) {
	// `fn` is just another reference to `foo`

	fn(); // <-- call-site!
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object

doFoo( obj.foo ); // "oops, global"
```

Parameter passing is just an implicit assignment, and since we're passing a function, it's an implicit reference assignment, so the end result is the same as the previous snippet.

参数传递仅仅是一个隐晦的赋值，一旦我们传递一个方法，这就是一个隐晦的引用赋值，所以最后的结果跟前面的那段代码一样。

What if the function you're passing your callback to is not your own, but built-in to the language? No difference, same outcome.

那如果将你的方法传入到一个语言内置的方法中而不是你自己的方法来作为回调会发生什么?没什么不一样，一样的结果。

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var a = "oops, global"; // `a` also property on global object

setTimeout( obj.foo, 100 ); // "oops, global"
```

Think about this crude theoretical pseudo-implementation of `setTimeout()` provided as a built-in from the JavaScript environment:

把这个粗略的理论上假装可执行的`setTimeout()`当做是JavaScript环境提供的内置方法一样：

```js
function setTimeout(fn,delay) {
	// wait (somehow) for `delay` milliseconds
	fn(); // <-- call-site!
}
```

It's quite common that our function callbacks *lose* their `this` binding, as we've just seen. But another way that `this` can surprise us is when the function we've passed our callback to intentionally changes the `this` for the call. Event handlers in popular JavaScript libraries are quite fond of forcing your callback to have a `this` which points to, for instance, the DOM element that triggered the event. While that may sometimes be useful, other times it can be downright infuriating. Unfortunately, these tools rarely let you choose.

就像我们看到的那样，我们的回调方法*丢失*他们的`this`绑定是非常常见的。`this`让我们吃惊的另外一种方式是当我们传入的回调方法有意的为这次调用改变了`this`。在流行的JavaScript库中事件处理非常乐于强行将你的回调方法`this`改变，举个例子，被触发事件的DOM节点。虽然有时候可能有用，其他时候会让人勃然大怒。不幸的是，这些工具很少会让你选择。

Either way the `this` is changed unexpectedly, you are not really in control of how your callback function reference will be executed, so you have no way (yet) of controlling the call-site to give your intended binding. We'll see shortly a way of "fixing" that problem by *fixing* the `this`.

不管哪一种`this`出乎意料的改变方式，你不能真正的控制你的回调方法引用会如何被执行，所以你没有办法(至少现在还没有)控制调用位置来给你进行有目的的绑定。我们将马上会看到一种方式通过*固定*`this`来"修复"这个问题.

### Explicit Binding

### 明确绑定

With *implicit binding* as we just saw, we had to mutate the object in question to include a reference on itself to the function, and use this property function reference to indirectly (implicitly) bind `this` to the object.

就像我们刚看到的*隐晦绑定*，我们需要转变一个对象在他本身包干一个方法的引用，然后使用这个方法属性间接的(隐晦的)绑定`this`为这个对象。

But, what if you want to force a function call to use a particular object for the `this` binding, without putting a property function reference on the object?

但是,如果你想强迫一个方法调用使用一个特定的对象作为`this`绑定如何在不用将一个属性方法引用放在对象上的方法达到目的？

"All" functions in the language have some utilities available to them (via their `[[Prototype]]` -- more on that later) which can be useful for this task. Specifically, functions have `call(..)` and `apply(..)` methods. Technically, JavaScript host environments sometimes provide functions which are special enough (a kind way of putting it!) that they do not have such functionality. But those are few. The vast majority of functions provided, and certainly all functions you will create, do have access to `call(..)` and `apply(..)`.

在语言中"所有"方法都有一些工具(经由他们的`[[Prototype]]` -- 更多的后面讲)可用于这个任务。明确的说,函数有`call(..)` 和 `apply(..)`方法。技术上来说，JavaScript环境有时候会提供足够特别方法(一种放上去方法)来实现他们没有的功能。但是这很少见。大多数方法被提供，事实上所有你将会创建的方法，他们都可以读取`call(..)` 和 `apply(..)`方法。

How do these utilities work? They both take, as their first parameter, an object to use for the `this`, and then invoke the function with that `this` specified. Since you are directly stating what you want the `this` to be, we call it *explicit binding*.

这些工具是如何工作的？他们都将第一个参数,一个对象来使用作为`this`，然后执行方法使用这个执行的`this`。一旦你直接指定你希望哪个作为`this`，我们叫做*明确绑定*。

Consider:

考虑一下

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

Invoking `foo` with *explicit binding* by `foo.call(..)` allows us to force its `this` to be `obj`.

通过`foo.call(..)`执行`foo`使用*明确绑定* 允许我们强行的`this`指向`obj`。

If you pass a simple primitive value (of type `string`, `boolean`, or `number`) as the `this` binding, the primitive value is wrapped in its object-form (`new String(..)`, `new Boolean(..)`, or `new Number(..)`, respectively). This is often referred to as "boxing".

如果你传入一个简单的原始值(`string`, `boolean`, 或者 `number`类型)作为`this`绑定，这个原始值会被他的对象包裹(各自,`new String(..)`, `new Boolean(..)`, 或者 `new Number(..)`)。这通常被叫做"装箱(boxing)"。

**Note:** With respect to `this` binding, `call(..)` and `apply(..)` are identical. They *do* behave differently with their additional parameters, but that's not something we care about presently.

**注意:** 就`this`绑定而言，`call(..)` 和 `apply(..)` 是相同的。但是他们对于他们附加的参数会有不同的行为，这个不是我们现在所要关心的。

Unfortunately, *explicit binding* alone still doesn't offer any solution to the issue mentioned previously, of a function "losing" its intended `this` binding, or just having it paved over by a framework, etc.

不幸的是，*明确绑定* 自己仍然没有提供任何解决方案用来解决我们之前提到过的问题，一个方法"丢失"他自己原本打算的`this`绑定，或者被一个框架覆盖,等等。

#### Hard Binding

#### 硬绑定

But a variation pattern around *explicit binding* actually does the trick. Consider:

但是*明确绑定*的一种变化形式的确可以完成这个戏法。考虑一下：

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

var bar = function() {
	foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` hard binds `foo`'s `this` to `obj`
// so that it cannot be overriden
bar.call( window ); // 2
```

Let's examine how this variation works. We create a function `bar()` which, internally, manually calls `foo.call(obj)`, thereby forcibly invoking `foo` with `obj` binding for `this`. No matter how you later invoke the function `bar`, it will always manually invoke `foo` with `obj`. This binding is both explicit and strong, so we call it *hard binding*.

让我们来探究下这个变化是如何工作的。我们创建一个方法`bar()`内部手动调用`foo.call(obj)`，此外执行`foo`强行将`this`绑定指向`obj`。不管你多晚执行方法`bar`，他都将会手动执行`foo`使用`obj`。这个绑定明确而且强壮，所以我们叫做 *硬绑定*.

The most typical way to wrap a function with a *hard binding* creates a pass-thru of any arguments passed and any return value received:

最典型的包裹一个方法使用*硬绑定*的方式是创建可以传入任何参数和接受任何返回值的通道。

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = function() {
	return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

Another way to express this pattern is to create a re-usable helper:

另外一种这个模式的表现是创建一个可重复使用的helper：

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

// simple `bind` helper
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}

var obj = {
	a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

Since *hard binding* is such a common pattern, it's provided with a built-in utility as of ES5: `Function.prototype.bind`, and it's used like this:

因为*硬绑定*是一种很常用的模式，ES5提供了内置的工具:`Function.prototype.bind`，它使用起来就像这样:

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = foo.bind( obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

`bind(..)` returns a new function that is hard-coded to call the original function with the `this` context set as you specified.

`bind(..)` 返回一个根据原始方法按照你执行的`this`上下文的硬编码的新方法。

**Note:** As of ES6, the hard-bound function produced by `bind(..)` has a `.name` property that derives from the original *target function*. For example: `bar = foo.bind(..)` should have a `bar.name` value of `"bound foo"`, which is the function call name that should show up in a stack trace.

**注意:** 在ES6中，`bind(..)`生产的硬绑定方法有一个`.name`属性源于原始的*目标方法*。举个例子:`bar = foo.bind(..)`会有一个`bar.name`值是`"bound foo"`，他是应该在调用跟踪里显示的方法调用名称。

#### API Call "Contexts"

#### API调用"上下文"

Many libraries' functions, and indeed many new built-in functions in the JavaScript language and host environment, provide an optional parameter, usually called "context", which is designed as a work-around for you not having to use `bind(..)` to ensure your callback function uses a particular `this`.

许多库的方法，许多新的JavaScript语言和宿主环境的内置方法，提供一个可选的参数，通常叫做"上下文"，设计用来在不需要使用`bind(..)`情况下确保你的回调用法使用一个指定的`this`。

For instance:

举个例子：

```js
function foo(el) {
	console.log( el, this.id );
}

var obj = {
	id: "awesome"
};

// use `obj` as `this` for `foo(..)` calls
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```

Internally, these various functions almost certainly use *explicit binding* via `call(..)` or `apply(..)`, saving you the trouble.

本质上，这些变化方法几乎可以确定经由`call(..)` 或者 `apply(..)`方法使用了*明确绑定*，拯救了你的麻烦。

### `new` Binding

### `new`绑定

The fourth and final rule for `this` binding requires us to re-think a very common misconception about functions and objects in JavaScript.

第四个也是最后一个`this`绑定规则需要我们重新回想一个关于在JavaScript中方法和对象非常常见的错误观念。

In traditional class-oriented languages, "constructors" are special methods attached to classes, that when the class is instantiated with a `new` operator, the constructor of that class is called. This usually looks something like:

在传统的的面向类的语言中，"构造器"是类的一个特殊方法，当类通过一个`new`得到实例化，类的构造方法会被调用。通常看起来是这样：

```js
something = new MyClass(..);
```

JavaScript has a `new` operator, and the code pattern to use it looks basically identical to what we see in those class-oriented languages; most developers assume that JavaScript's mechanism is doing something similar. However, there really is *no connection* to class-oriented functionality implied by `new` usage in JS.

JavaScript也有一个`new`操作符，代码模式基本上看起来和那些面向类的语言差不多；许多开发者会假设JavaScript的机制会做相似的操作。然而，在JS中使用`new`关键字看起来暗示和面向类的功能方法但是和面向类的功能方法是真的"没有联系"。

First, let's re-define what a "constructor" in JavaScript is. In JS, constructors are **just functions** that happen to be called with the `new` operator in front of them. They are not attached to classes, nor are they instantiating a class. They are not even special types of functions. They're just regular functions that are, in essence, hijacked by the use of `new` in their invocation.

首先，来重新定义一下在JavaScript中什么是"构造器"。在JS中，构造器**仅仅是方法**调用发生在在他们前面使用`new`操作调用时，他们不依附于类，也不会实例化一个类。他们甚至不是一个特殊类型的方法。他们只是平常的方法，本质上，是通过在他们执行时使用`new`来进行操作。

For example, the `Number(..)` function acting as a constructor, quoting from the ES5.1 spec:

举个例子，`Number(..)`方法表现起来就像一个构造器，引用ES5.1的描述:

> 15.7.2 The Number Constructor

> 15.7.2 数字构造器
>
> When Number is called as part of a new expression it is a constructor: it initialises the newly created object.

> 当数字被作为一个new表达式的一部分被调用时这就是一个构造器:初始化创造一个新的对象。

So, pretty much any ol' function, including the built-in object functions like `Number(..)` (see Chapter 3) can be called with `new` in front of it, and that makes that function call a *constructor call*. This is an important but subtle distinction: there's really no such thing as "constructor functions", but rather construction calls *of* functions.

所以，许多方法,包括内置的对象方法像`Number(..)`(见第三章)可以在前面使用`new`调用，这样使方法变成一个*构造器调用*。这里有一个重要但是微妙的细节:他们实际上没有类似像"构造方法"，但是有函数的构造调用。

When a function is invoked with `new` in front of it, otherwise known as a constructor call, the following things are done automatically:

当一个方法在前面加上`new`被调用，被称作一个构造器调用，下面的事情将会自动执行：

1. a brand new object is created (aka, constructed) out of thin air
1. 一个崭新的对象会凭空被创建(又称作被构造)
2. *the newly constructed object is `[[Prototype]]`-linked*
2. *新的被构造的对象会被连接上`[[Prototype]]`*
3. the newly constructed object is set as the `this` binding for that function call
3. 新的被构造的对象会作为方法调用的`this`绑定
4. unless the function returns its own alternate **object**, the `new`-invoked function call will *automatically* return the newly constructed object.
4. 除非方法返回它自己的**对象**，否则`new`新的被执行的方法调用会*自动*返回新的被构造完的对象。

Steps 1, 3, and 4 apply to our current discussion. We'll skip over step 2 for now and come back to it in Chapter 5.

第1,3和4步适用于我们现在的讨论，我们现在将会跳过第二步在第五章在回来讨论他。

Consider this code:

考虑一下这个代码：

```js
function foo(a) {
	this.a = a;
}

var bar = new foo( 2 );
console.log( bar.a ); // 2
```

By calling `foo(..)` with `new` in front of it, we've constructed a new object and set that new object as the `this` for the call of `foo(..)`. **So `new` is the final way that a function call's `this` can be bound.** We'll call this *new binding*.

通过把`new`放在`foo(..)`之前调用方法，我们构造得到一个新的对象然后设置这个新的对象作为`foo(..)`调用的`this`绑定。**所以`new`是最后一种将一个方法的`this`绑定的方式** 我们称之为*new绑定*。

## Everything In Order

## 所有的排序

So, now we've uncovered the 4 rules for binding `this` in function calls. *All* you need to do is find the call-site and inspect it to see which rule applies. But, what if the call-site has multiple eligible rules? There must be an order of precedence to these rules, and so we will next demonstrate what order to apply the rules.

所以，现在我们揭开了`this`绑定在方法调用中的4种规则。你需要做的仅仅是找到调用位置然后检查哪种规则被应用。但是，如果调用位置有第一个适用规则怎么办？我们必须给这些规则按优先级排一下序，所以我们将接下来说明这些规则应用的顺序。

It should be clear that the *default binding* is the lowest priority rule of the 4. So we'll just set that one aside.

应该要清楚*默认绑定*是4种规则中优先级最低的。说我们会先将他放在一边。

Which is more precedent, *implicit binding* or *explicit binding*? Let's test it:

哪个在更前面，*隐晦绑定* 或是 *明确绑定* ?让我们测试以下:

```js
function foo() {
	console.log( this.a );
}

var obj1 = {
	a: 2,
	foo: foo
};

var obj2 = {
	a: 3,
	foo: foo
};

obj1.foo(); // 2
obj2.foo(); // 3

obj1.foo.call( obj2 ); // 3
obj2.foo.call( obj1 ); // 2
```

So, *explicit binding* takes precedence over *implicit binding*, which means you should ask **first** if *explicit binding* applies before checking for *implicit binding*.

所以，*明确绑定* 比 *隐晦绑定* 有更高的优先权，这意味着你应该检查*隐晦绑定*之前**先**检查*明确绑定*是否被适用。

Now, we just need to figure out where *new binding* fits in the precedence.

现在，我们只需要搞清楚*new绑定*在优先顺序中的合适位置就可以了。

```js
function foo(something) {
	this.a = something;
}

var obj1 = {
	foo: foo
};

var obj2 = {};

obj1.foo( 2 );
console.log( obj1.a ); // 2

obj1.foo.call( obj2, 3 );
console.log( obj2.a ); // 3

var bar = new obj1.foo( 4 );
console.log( obj1.a ); // 2
console.log( bar.a ); // 4
```

OK, *new binding* is more precedent than *implicit binding*. But do you think *new binding* is more or less precedent than *explicit binding*?

OK,*new绑定* 比 *隐晦绑定* 有更高的优先权。但是你觉得*new绑定*比*明确绑定*的优先权是高还是低呢?

**Note:** `new` and `call`/`apply` cannot be used together, so `new foo.call(obj1)` is not allowed, to test *new binding* directly against *explicit binding*. But we can still use a *hard binding* to test the precedence of the two rules.

**注意:**`new`和`call`/`apply`不可以在一起使用,所以`new foo.call(obj1)`是不被允许的，所以无法直接测试*new绑定*和*明确绑定*。但是我们仍然可以使用一个*硬绑定*来测试这两种规则的优先权。

Before we explore that in a code listing, think back to how *hard binding* physically works, which is that `Function.prototype.bind(..)` creates a new wrapper function that is hard-coded to ignore its own `this` binding (whatever it may be), and use a manual one we provide.

在我们在代码中探索之前，回想一下*硬绑定*是如何工作的，`Function.prototype.bind(..)`创建一个新的包围方法强行的忽略本身的`this`绑定(不论是否存在)，然后使用一个我们手工提供的一个。

By that reasoning, it would seem obvious to assume that *hard binding* (which is a form of *explicit binding*) is more precedent than *new binding*, and thus cannot be overridden with `new`.

因为这个原因，显然可以假设*硬绑定* (*明确绑定* 的一种类型)比*new绑定*的优先级更高，因此不能被new覆盖。

Let's check:

让我们检查一下:

```js
function foo(something) {
	this.a = something;
}

var obj1 = {};

var bar = foo.bind( obj1 );
bar( 2 );
console.log( obj1.a ); // 2

var baz = new bar( 3 );
console.log( obj1.a ); // 2
console.log( baz.a ); // 3
```

Whoa! `bar` is hard-bound against `obj1`, but `new bar(3)` did **not** change `obj1.a` to be `3` as we would have expected. Instead, the *hard bound* (to `obj1`) call to `bar(..)` ***is*** able to be overridden with `new`. Since `new` was applied, we got the newly created object back, which we named `baz`, and we see in fact that  `baz.a` has the value `3`.

哇哦！`bar`紧紧的强行绑定住`obj1`，但是`new bar(3)`事实上**没有**像我们期望的那样将`obj1.a`修改为`3`。相反的，`bar(..)`调用的*硬绑定*(对`obj1`的)***是*** 可以通过`new`覆盖的。一旦`new`被应用，我们获得了一个新创建的对象，我们取名叫`baz`，然后我们看到事实上`baz.a`的值是`3`。

This should be surprising if you go back to our "fake" bind helper:

如果你回过头来看我们之前"假的"绑定helper你应该会吃惊：

```js
function bind(fn, obj) {
	return function() {
		fn.apply( obj, arguments );
	};
}
```

If you reason about how the helper's code works, it does not have a way for a `new` operator call to override the hard-binding to `obj` as we just observed.

如果你想知道helper的代码是如何工作的原因，这不会有像我们观察到的的一样的方式用一个`new`调用来覆盖硬绑定的`obj`。

But the built-in `Function.prototype.bind(..)` as of ES5 is more sophisticated, quite a bit so in fact. Here is the (slightly reformatted) polyfill provided by the MDN page for `bind(..)`:

但是ES5内置的`Function.prototype.bind(..)`更复杂，事实上复杂的多。这里是(稍微重新格式化了一下)MDN提供的`bind(..)`的polyfill：

```js
if (!Function.prototype.bind) {
	Function.prototype.bind = function(oThis) {
		if (typeof this !== "function") {
			// closest thing possible to the ECMAScript 5
			// internal IsCallable function
			throw new TypeError( "Function.prototype.bind - what " +
				"is trying to be bound is not callable"
			);
		}

		var aArgs = Array.prototype.slice.call( arguments, 1 ),
			fToBind = this,
			fNOP = function(){},
			fBound = function(){
				return fToBind.apply(
					(
						this instanceof fNOP &&
						oThis ? this : oThis
					),
					aArgs.concat( Array.prototype.slice.call( arguments ) )
				);
			}
		;

		fNOP.prototype = this.prototype;
		fBound.prototype = new fNOP();

		return fBound;
	};
}
```

**Note:** The `bind(..)` polyfill shown above differs from the built-in `bind(..)` in ES5 with respect to hard-bound functions that will be used with `new` (see below for why that's useful). Because the polyfill cannot create a function without a `.prototype` as the built-in utility does, there's some nuanced indirection to approximate the same behavior. Tread carefully if you plan to use `new` with a hard-bound function and you rely on this polyfill.

**注意:** `bind(..)`polyfill和ES5内置的`bind(..)`在方法硬绑定上使用了不一样的方式,这个polyfill使用了`new`(为什么这么用见下面)。因为polyfill不能像内置方法一样在不使用一个`.prototype`的情况下创建一个方法，使用一些微妙间接的方式来达到相似的行为。如果你计划使用`new`来做方法硬绑定而且依赖这个polyfill那你就需要小心对待了。

The part that's allowing `new` overriding is:

这个部分允许`new`覆盖是:

```js
this instanceof fNOP &&
oThis ? this : oThis

// ... and:

fNOP.prototype = this.prototype;
fBound.prototype = new fNOP();
```

We won't actually dive into explaining how this trickery works (it's complicated and beyond our scope here), but essentially the utility determines whether or not the hard-bound function has been called with `new` (resulting in a newly constructed object being its `this`), and if so, it uses *that* newly created `this` rather than the previously specified *hard binding* for `this`.

我们事实上不是深究去解释这个把戏是如何工作的(这个跟作用域有关并且比我们这里讨论的复杂),但是事实上这个工具判断硬绑定是否通过`new`(导致一个新的被初始化的对象变成了`this`)来调用，如果是的话，他会使用那个新创建的 `this` 而不是之前*硬绑定*指定的`this`。

Why is `new` being able to override *hard binding* useful?

为什么`new`可以覆盖*硬绑定*那么有用?

The primary reason for this behavior is to create a function (that can be used with `new` for constructing objects) that essentially ignores the `this` *hard binding* but which presets some or all of the function's arguments. One of the capabilities of `bind(..)` is that any arguments passed after the first `this` binding argument are defaulted as standard arguments to the underlying function (technically called "partial application", which is a subset of "currying").

这个行为有用的主要原因是创建一个方法(他可以通过使用`new`来创建对象)他本质上可以忽略预先设置了一些或者全部方法参数的*硬绑定*的`this`.`bind(..)`的其中一个能力是在第一个`this`绑定参数的后面的任何一个被传入的参数默认作为后面方法的标准参数(技术上称作"部分应用"，他是一种"currying").

For example:

举个例子:

```js
function foo(p1,p2) {
	this.val = p1 + p2;
}

// using `null` here because we don't care about
// the `this` hard-binding in this scenario, and
// it will be overridden by the `new` call anyway!
var bar = foo.bind( null, "p1" );

var baz = new bar( "p2" );

baz.val; // p1p2
```

### Determining `this`

### 查明`this`

Now, we can summarize the rules for determining `this` from a function call's call-site, in their order of precedence. Ask these questions in this order, and stop when the first rule applies.

现在，我们可以总结从一个方法的调用位置如何查明他应用了哪个`this`规则，根据他们的优先顺序。根据他们顺序提出这些问题，第一个应用规则找到时停止。

1. Is the function called with `new` (**new binding**)? If so, `this` is the newly constructed object.

1. 方法是否通过`new`调用(**new绑定**)?如果是的话, `this`是新的构建完的对象。

    `var bar = new foo()`

2. Is the function called with `call` or `apply` (**explicit binding**), even hidden inside a `bind` *hard binding*? If so, `this` is the explicitly specified object.

2. 方法是否通过`call` 或者 `apply`调用(**明确绑定**),或者是隐藏在`bind`之中 *硬绑定*?如果是的话, `this`是明确指定的对象。

    `var bar = foo.call( obj2 )`

3. Is the function called with a context (**implicit binding**), otherwise known as an owning or containing object? If so, `this` is *that* context object.

3. 方法是否通过上下文调用(**隐含绑定**),亦或者是他自己或者包含的对象?如果是的话,`this`是那个上下文对象.

    `var bar = obj1.foo()`

4. Otherwise, default the `this` (**default binding**). If in `strict mode`, pick `undefined`, otherwise pick the `global` object.

4. 最后,默认的`this`(**默认绑定**).如果是在`strict mode`(严格模式)下,选取`undefined`值，否则的话会选取`global`对象。

    `var bar = foo()`

That's it. That's *all it takes* to understand the rules of `this` binding for normal function calls. Well... almost.

这个就是需要了解普通方法调用的 `this` 绑定规则的全部东西. Well...几乎是。

## Binding Exceptions

## 绑定特例

As usual, there are some *exceptions* to the "rules".

通常情况下,这些"规则"有一些特例.

The `this`-binding behavior can in some scenarios be surprising, where you intended a different binding but you end up with binding behavior from the *default binding* rule (see previous).

`this`绑定的举止可以和一些预先设想好的不太一样令人吃惊，你可能获得一个*默认绑定*规则的绑定行为这个和你原本打算的绑定不太一样(见前面).

### Ignored `this`

### 忽略`this`

If you pass `null` or `undefined` as a `this` binding parameter to `call`, `apply`, or `bind`, those values are effectively ignored, and instead the *default binding* rule applies to the invocation.

如果你将`null` or `undefined`作为一个`this`绑定参数传入到 `call`, `apply`, 或者 `bind`中，这些值会被忽略，取而代之的是*默认绑定*规则被这次执行所应用。

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo.call( null ); // 2
```

Why would you intentionally pass something like `null` for a `this` binding?

为什么你有意的传入一些类似`null`的值作为一个`this`绑定?

It's quite common to use `apply(..)` for spreading out arrays of values as parameters to a function call. Similarly, `bind(..)` can curry parameters (pre-set values), which can be very helpful.

在使用`apply(..)`时将一个值的平铺数组作为参数传入一个方法调用时很常见的。类似的，`bind(..)`可以curry参数(预先设置参数)，这可以非常有用。

```js
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// spreading out array as parameters
foo.apply( null, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

Both these utilities require a `this` binding for the first parameter. If the functions in question don't care about `this`, you need a placeholder value, and `null` might seem like a reasonable choice as shown in this snippet.

这些工具都需要一个`this`绑定作为第一个参数。如果方法不在乎什么作为`this`，你需要一个占位值，在这个代码片段中`null`看起来似乎像一个合理的选择。

**Note:** We don't cover it in this book, but ES6 has the `...` spread operator which will let you syntactically "spread out" an array as parameters without needing `apply(..)`, such as `foo(...[1,2])`, which amounts to `foo(1,2)` -- syntactically avoiding a `this` binding if it's unnecessary. Unfortunately, there's no ES6 syntactic substitute for currying, so the `this` parameter of the `bind(..)` call still needs attention.

**注意:** ES6中有`...`的铺开操作他可以让你按照语法的在不需要`apply(..)`的情况下"展开"一个数组作为参数，就像`foo(...[1,2])`，他会展开成为`foo(1,2)` -- 语法上在不是必须的情况下避免一个 `this` 绑定。遗憾的是，ES6语法中没有用于替代克里化的，所以`bind(..)`调用的 的`this` 参数仍然需要注意。

However, there's a slight hidden "danger" in always using `null` when you don't care about the `this` binding. If you ever use that against a function call (for instance, a third-party library function that you don't control), and that function *does* make a `this` reference, the *default binding* rule means it might inadvertently reference (or worse, mutate!) the `global` object (`window` in the browser).

然而,当你不在意`this`绑定时经常使用`null`会一个隐藏的小问题。当你在方法中这样使用(举例来说,一个你没办法控制的第三方库的方法)，而且这个方法使用了一个 `this` 引用，*默认绑定* 规则会无意(或者更糟糕)的引用`global`对象(在浏览器中为`window`)。

Obviously, such a pitfall can lead to a variety of *very difficult* to diagnose/track-down bugs.

显然，这样陷阱会导致各种各样非常难以诊断/跟踪的bug。

#### Safer `this`

#### 更安全的`this`

Perhaps a somewhat "safer" practice is to pass a specifically set up object for `this` which is guaranteed not to be an object that can create problematic side effects in your program. Borrowing terminology from networking (and the military), we can create a "DMZ" (de-militarized zone) object -- nothing more special than a completely empty, non-delegated (see Chapters 5 and 6) object.

也许某种"更安全"的实践是传一个已经设置好的执行的对象作为`this`这样可以保证这个对象不会创造问题来影响你的程序。从网络(和军事)上借用一个专用名词,我们可以创建一个"DMZ"(安全区域)对象 -- 一个平淡无奇的完全空,没有委托(见第5章和第6章)的对象。

If we always pass a DMZ object for ignored `this` bindings we don't think we need to care about, we're sure any hidden/unexpected usage of `this` will be restricted to the empty object, which insulates our program's `global` object from side-effects.

如果我们认为我们不需要在意一个方法的`this`绑定时始终传入一个DMZ对象来忽略`this` bindings，我们可以确定任何隐藏/无意的使用`this`将会受到空对象的约束，这样可以免除我们程序`global`对象的影响。

Since this object is totally empty, I personally like to give it the variable name `ø` (the lowercase mathematical symbol for the empty set). On many keyboards (like US-layout on Mac), this symbol is easily typed with `⌥`+`o` (option+`o`). Some systems also let you set up hotkeys for specific symbols. If you don't like the `ø` symbol, or your keyboard doesn't make that as easy to type, you can of course call it whatever you want.

一旦对象完全为空，我个人喜欢给一个变量名称`ø`(小写的数学符号代表空集)。在很多键盘上(像MAC上的US-layout),这个符号可以使用`⌥`+`o` (option+`o`)很简单的打出来。一些系统也可以让你为这个特殊符号设置热键。如果你不喜欢`ø`符号,或者你的键盘不能让很简单的打出来，你当然可以取任何你想要的名字。

Whatever you call it, the easiest way to set it up as **totally empty** is `Object.create(null)` (see Chapter 5). `Object.create(null)` is similar to `{ }`, but without the delegation to `Object.prototype`, so it's "more empty" than just `{ }`.

不论你叫他声明，设置一个**完全为空**最简单的方式是`Object.create(null)`(见第五章).`Object.create(null)`和`{ }`相类似,但是`Object.create(null)`没有`Object.prototype`的委托，所以这个比`{ }`来的更空。

```js
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// our DMZ empty object
var ø = Object.create( null );

// spreading out array as parameters
foo.apply( ø, [2, 3] ); // a:2, b:3

// currying with `bind(..)`
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

Not only functionally "safer", there's a sort of stylistic benefit to `ø`, in that it semantically conveys "I want the `this` to be empty" a little more clearly than `null` might. But again, name your DMZ object whatever you prefer.

不仅仅是方法上的"更安全"，这对代码风格上`ø`也有益处，这个从语义上传达"我想要`this`完全为空"这比`null`看起来更清楚。但是在说一遍，只要你觉得更好随便怎么样给你的DMZ对象取名。

### Indirection

### 间接

Another thing to be aware of is you can (intentionally or not!) create "indirect references" to functions, and in those cases,  when that function reference is invoked, the *default binding* rule also applies.

另外一个需要知道的是你可以(有意或者无意)给方法创建"间接引用"，在这些情况下，当那个方法引用被执行，*默认绑定* 规则也被引用。

One of the most common ways that *indirect references* occur is from an assignment:

*间接引用* 发生最常见的的一种方式是一个赋值表达式:

```js
function foo() {
	console.log( this.a );
}

var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };

o.foo(); // 3
(p.foo = o.foo)(); // 2
```

The *result value* of the assignment expression `p.foo = o.foo` is a reference to just the underlying function object. As such, the effective call-site is just `foo()`, not `p.foo()` or `o.foo()` as you might expect. Per the rules above, the *default binding* rule applies.

赋值表达式`p.foo = o.foo`的结果值是一个隐含的方法对象的引用。因为这样,有效的调用位置是`foo()`，不是你预期中的 `p.foo()` 或者 `o.foo()`。根据上面提到的规则，*默认绑定* 规则被引用。

Reminder: regardless of how you get to a function invocation using the *default binding* rule, the `strict mode` status of the **contents** of the invoked function making the `this` reference -- not the function call-site -- determines the *default binding* value: either the `global` object if in non-`strict mode` or `undefined` if in `strict mode`.

提醒：不论你怎么得到引用了*默认绑定* 规则的方法调用，被调用的内容的`strict mode`(严格模式)状态决定了`this`引用 -- 不是方法调用位置 -- 查明*默认绑定*的值:如果是非`strict mode`就是`global` 对象如果是`strict mode`就是 `undefined`。

### Softening Binding

### 软绑定

We saw earlier that *hard binding* was one strategy for preventing a function call falling back to the *default binding* rule inadvertently, by forcing it to be bound to a specific `this` (unless you use `new` to override it!). The problem is, *hard-binding* greatly reduces the flexibility of a function, preventing manual `this` override with either the *implicit binding* or even subsequent *explicit binding* attempts.

我们看见之前的*硬绑定*是一种防止方法调用在无意的情况下退回到使用*默认绑定*的策略，强迫他绑定一个指定的 `this` (至少你使用`new`去覆盖他)。问题是，*硬绑定* 大大的约束了方法的灵活性，预防手动的 `this`覆盖不论是*隐晦绑定*或者是随后的试图*明确绑定*。

It would be nice if there was a way to provide a different default for *default binding* (not `global` or `undefined`), while still leaving the function able to be manually `this` bound via *implicit binding* or *explicit binding* techniques.

如果有一种方式提供一种不一样的*默认绑定*方式(不是`global` 或者 `undefined`)这将会很棒，同时也可以让方法经由*隐晦绑定*或者*明确绑定*技术手动绑定`this`。

We can construct a so-called *soft binding* utility which emulates our desired behavior.

我们可以构建一个称作*软绑定*的工具来模仿我们想要得到的行为。

```js
if (!Function.prototype.softBind) {
	Function.prototype.softBind = function(obj) {
		var fn = this,
			curried = [].slice.call( arguments, 1 ),
			bound = function bound() {
				return fn.apply(
					(!this ||
						(typeof window !== "undefined" &&
							this === window) ||
						(typeof global !== "undefined" &&
							this === global)
					) ? obj : this,
					curried.concat.apply( curried, arguments )
				);
			};
		bound.prototype = Object.create( fn.prototype );
		return bound;
	};
}
```

The `softBind(..)` utility provided here works similarly to the built-in ES5 `bind(..)` utility, except with our *soft binding* behavior. It wraps the specified function in logic that checks the `this` at call-time and if it's `global` or `undefined`, uses a pre-specified alternate *default* (`obj`). Otherwise the `this` is left untouched. It also provides optional currying (see the `bind(..)` discussion earlier).

`softBind(..)`工具提供的工作方式类似于ES5内置的`bind(..)` 工具，除了我们的*软绑定*行为。他包括了指定方法在调用时逻辑上检查`this`如果是`global` 或者 `undefined`，用一个预先各自指定的*默认*(`obj`)。否则`this`就不做改变。他也提供了合适的currying(见我们之前讨论的`bind(..)`)。

Let's demonstrate its usage:

让我们来演示他的用法：

```js
function foo() {
   console.log("name: " + this.name);
}

var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };

var fooOBJ = foo.softBind( obj );

fooOBJ(); // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2   <---- look!!!

fooOBJ.call( obj3 ); // name: obj3   <---- look!

setTimeout( obj2.foo, 10 ); // name: obj   <---- falls back to soft-binding
```

The soft-bound version of the `foo()` function can be manually `this`-bound to `obj2` or `obj3` as shown, but it falls back to `obj` if the *default binding* would otherwise apply.

软绑定版本的`foo()`方法可以像看到的那样手动绑定 `this`到`obj2` 或者 `obj3`，同时如果是*默认绑定*应用也可以退回到`obj`。

## Lexical `this`

## 词法`this`

Normal functions abide by the 4 rules we just covered. But ES6 introduces a special kind of function that does not use these rules: arrow-function.

普通方法遵循4种规则我们已经都讲了。但是ES6介绍了一种特别的方法不能使用这些规则:箭头方法。

Arrow-functions are signified not by the `function` keyword, but by the `=>` so called "fat arrow" operator. Instead of using the four standard `this` rules, arrow-functions adopt the `this` binding from the enclosing (function or global) scope.

箭头方法声明不是使用`function`关键字，使用`=>`所以被称作"箭头"操作符。使用4种标准的规则不同的是，箭头方法从包围的(方法或者全局)作用域接受`this`绑定。

Let's illustrate arrow-function lexical scope:

让我们距离说明一下箭头方法的词法作用域:

```js
function foo() {
	// return an arrow function
	return (a) => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	};
}

var obj1 = {
	a: 2
};

var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, not 3!
```

The arrow-function created in `foo()` lexically captures whatever `foo()`s `this` is at its call-time. Since `foo()` was `this`-bound to `obj1`, `bar` (a reference to the returned arrow-function) will also be `this`-bound to `obj1`. The lexical binding of an arrow-function cannot be overridden (even with `new`!).

箭头函数在`foo()` 词法作用域中被创建捕获 `foo()`的`this`在他执行的时候。一旦 `foo()` 绑定将`this`到`obj1`，`bar`(一个箭头函数被返回)将也会将`this`绑定到 `obj1`。箭头函数的词法绑定不能被重写(即使是使用`new`!)。

The most common use-case will likely be in the use of callbacks, such as event handlers or timers:

最常见的使用场景是使用在回调中，类似事件控制器或者事件：

```js
function foo() {
	setTimeout(() => {
		// `this` here is lexically adopted from `foo()`
		console.log( this.a );
	},100);
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

While arrow-functions provide an alternative to using `bind(..)` on a function to ensure its `this`, which can seem attractive, it's important to note that they essentially are disabling the traditional `this` mechanism in favor of more widely-understood lexical scoping. Pre-ES6, we already have a fairly common pattern for doing so, which is basically almost indistinguishable from the spirit of ES6 arrow-functions:

箭头函数提供了一个可以替代在方法上使用`bind(..)`的方式来确保`this`，这看起来挺招人喜欢，需要重点注意的是他本质上禁用了传统的`this`机制而采用了更广为人知的词法作用域。ES6之前我们已经有了相当普通的模式来这样做，基本上几乎和ES6的箭头方法的精神一样：

```js
function foo() {
	var self = this; // lexical capture of `this`
	setTimeout( function(){
		console.log( self.a );
	}, 100 );
}

var obj = {
	a: 2
};

foo.call( obj ); // 2
```

While `self = this` and arrow-functions both seem like good "solutions" to not wanting to use `bind(..)`, they are essentially fleeing from `this` instead of understanding and embracing it.

而`self = this` 和箭头函数在你不想使用 `bind(..)`的时候看起来都像是好的"解决方案"，他们本质上是逃避`this`而不是理解和拥抱他。

If you find yourself writing `this`-style code, but most or all the time, you defeat the `this` mechanism with lexical `self = this` or arrow-function "tricks", perhaps you should either:

如果你发现你自己写`this`风格的代码，但是大多数或者全部时候，你使用词法`self = this` 或者箭头函数"把戏"来击败 `this`机制，也许你也可以选择:

1. Use only lexical scope and forget the false pretense of `this`-style code.

1. 只使用词法作用域然后忘掉`this`风格代码的错误模式.

2. Embrace `this`-style mechanisms completely, including using `bind(..)` where necessary, and try to avoid `self = this` and arrow-function "lexical this" tricks.

2. 完全拥抱`this`风格的机制，包括在需要的时候使用`bind(..)`，然后尝试避免`self = this`和箭头函数"词法this"的把戏。

A program can effectively use both styles of code (lexical and `this`), but inside of the same function, and indeed for the same sorts of look-ups, mixing the two mechanisms is usually asking for harder-to-maintain code, and probably working too hard to be clever.

一个程序可以有效的同时使用两种风格的代码(词法和`this`)，但是在同一个方法的内部，和同种类型的查看，混合两种机制通常会带来难以维护的代码，很有可能是工作变的艰难。

## Review (TL;DR)

## 回顾(TL;DR)

Determining the `this` binding for an executing function requires finding the direct call-site of that function. Once examined, four rules can be applied to the call-site, in *this* order of precedence:

一个执行的方法查明`this`绑定需要找到方法的调用位置。一个个的检查，4个规则可以被应用于调用位置，他们有这样的优先顺序:

1. Called with `new`? Use the newly constructed object.

1. 通过`new`调用？用新创建的对象。

2. Called with `call` or `apply` (or `bind`)? Use the specified object.

2. 通过 `call` 或者 `apply` (或者 `bind`)调用？使用指定的对象。

3. Called with a context object owning the call? Use that context object.

3. 通过一个上下文对象自己调用?使用这个上下文对象。

4. Default: `undefined` in `strict mode`, global object otherwise.

4. 默认:`strict mode`(严格模式)下为`undefined`,此外是global对象。

Be careful of accidental/unintentional invoking of the *default binding* rule. In cases where you want to "safely" ignore a `this` binding, a "DMZ" object like `ø = Object.create(null)` is a good placeholder value that protects the `global` object from unintended side-effects.

有意/无意执行*默认绑定*时要小心。当你想"更安全"的忽略一个 `this`绑定的情况下，一个"DMZ"对象类似`ø = Object.create(null)` 是一个好的占位值他可以保护无意间使用`global`对象带来的不好的影响。

Instead of the four standard binding rules, ES6 arrow-functions use lexical scoping for `this` binding, which means they adopt the `this` binding (whatever it is) from its enclosing function call. They are essentially a syntactic replacement of `self = this` in pre-ES6 coding.

和4种标准的绑定规则不同，ES6箭头函数使用词法作用域作为`this`绑定，这意味着他们从包围的方法调用中获取`this`绑定。在ES6之前这本质上在可以通过 `self = this`这种句法上来替代。
