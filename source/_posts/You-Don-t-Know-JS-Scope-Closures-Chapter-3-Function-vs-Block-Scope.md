---
layout: post
title: 'You Don''t Know JS: Scope & Closures-Chapter 3: Function vs. Block Scope'
date: 2017-07-29 12:16:13
tags:
---

# You Don't Know JS: Scope & Closures
# Chapter 3: Function vs. Block Scope

# 你不知道的JS:作用域和闭包
# 第三章:方法 vs. 作用域块

As we explored in Chapter 2, scope consists of a series of "bubbles" that each act as a container or bucket, in which identifiers (variables, functions) are declared. These bubbles nest neatly inside each other, and this nesting is defined at author-time.

根据我们在第二章探讨的，作用域由一系列的"气泡"组成每个充当容器或者桶，内部有识别符(变量,方法)被声明。这些气泡有序的嵌套着其他气泡，这些嵌套关系在书写时间被定义。

But what exactly makes a new bubble? Is it only the function? Can other structures in JavaScript create bubbles of scope?

但是到底是什么创造了新的气泡？只是方法？是否有其他构造在JavaScript里可以创造作用域气泡？

## Scope From Functions

## 方法作用域

The most common answer to those questions is that JavaScript has function-based scope. That is, each function you declare creates a bubble for itself, but no other structures create their own scope bubbles. As we'll see in just a little bit, this is not quite true.

对于这些问题最常见的回答是JavaScript是基于方法的作用域。就因为这样，每个你声明的方法就为他自己创建一个气泡，但是没有其他结构可以创建他们自己的作用域气泡。但是我们将马上会看到,这不是完全正确的。

But first, let's explore function scope and its implications.

但是首先,我们探索方法作用域和他的含义。

Consider this code:

考虑这段代码：

```js
function foo(a) {
	var b = 2;

	// some code

	function bar() {
		// ...
	}

	// more code

	var c = 3;
}
```

In this snippet, the scope bubble for `foo(..)` includes identifiers `a`, `b`, `c` and `bar`. **It doesn't matter** *where* in the scope a declaration appears, the variable or function belongs to the containing scope bubble, regardless. We'll explore how exactly *that* works in the next chapter.

在这个片段中,`foo(..)`气泡作用域包含了标识符`a`, `b`, `c` 和 `bar`。不管作用域在 *哪里* 出现声明**这都没关系**,变量或者方法属于包含他的作用域气泡。我们会在下一章探索 *那* 到底是如何工作的。

`bar(..)` has its own scope bubble. So does the global scope, which has just one identifier attached to it: `foo`.

`bar(..)`有他自己的作用域气泡。全局作用域也是如此,他只有一个标识符属于他：`foo`。

Because `a`, `b`, `c`, and `bar` all belong to the scope bubble of `foo(..)`, they are not accessible outside of `foo(..)`. That is, the following code would all result in `ReferenceError` errors, as the identifiers are not available to the global scope:

因为`a`, `b`, `c`和`bar`都属于作用域`foo(..)`，`foo(..)`外部是无法访问的。所以接下来的代码会都会抛出`ReferenceError(引用错误)`错误，因为标识符对于全局作用域是不可获得的：

```js
bar(); // fails

console.log( a, b, c ); // all 3 fail
```

However, all these identifiers (`a`, `b`, `c`, `foo`, and `bar`) are accessible *inside* of `foo(..)`, and indeed also available inside of `bar(..)` (assuming there are no shadow identifier declarations inside `bar(..)`).

然而，所有的标识符(`a`, `b`, `c`, `foo`, 和 `bar`)在`foo(..)`里都是可以访问的，在`bar(..)`内部实际上也可以访问(假设在`bar(..)`内部没有遮蔽的识别符声明)。

Function scope encourages the idea that all variables belong to the function, and can be used and reused throughout the entirety of the function (and indeed, accessible even to nested scopes). This design approach can be quite useful, and certainly can make full use of the "dynamic" nature of JavaScript variables to take on values of different types as needed.

方法作用域鼓励的想法是(encourages:to persuade sb to do sth by making it easier for them and making them believe it is a good thing to do)所有的变量属于方法，变量可以贯穿整个方法使用和重用(事实上,在被嵌套的方法中也可以访问)。整个设计将会变得十分有用，肯定可以充分利用"动态"原始JavaScript变量来根据不同的需要的类型获取值。

On the other hand, if you don't take careful precautions, variables existing across the entirety of a scope can lead to some unexpected pitfalls.

其他方面来说，如果你不做预防，变量穿过这个变量会带来不可估计的陷阱。

## Hiding In Plain Scope

## 隐藏在普通的作用域

The traditional way of thinking about functions is that you declare a function, and then add code inside it. But the inverse thinking is equally powerful and useful: take any arbitrary section of code you've written, and wrap a function declaration around it, which in effect "hides" the code.

对方法传统的看法是声明一个方法，然后在里面加代码。但是相反的现在的看法趋向有力和有用:取任意部分你写的代码，然后声明一个方法把它包起来，有效的把代码"隐藏"起来。

The practical result is to create a scope bubble around the code in question, which means that any declarations (variable or function) in that code will now be tied to the scope of the new wrapping function, rather than the previously enclosing scope. In other words, you can "hide" variables and functions by enclosing them in the scope of a function.

其实际结果是在代码周围创建了一个作用域气泡，意思是方法中任何的声明(方法或者变量)将会被新包围的方法的作用域约束，也不是先前包围的作用域。换句话来说，你可以将变量和方法用方法的作用域包围"藏"起来。

Why would "hiding" variables and functions be a useful technique?

为什么"隐藏"变量和方法会是有用的技巧？

There's a variety of reasons motivating this scope-based hiding. They tend to arise from the software design principle "Principle of Least Privilege" [^note-leastprivilege], also sometimes called "Least Authority" or "Least Exposure". This principle states that in the design of software, such as the API for a module/object, you should expose only what is minimally necessary, and "hide" everything else.

有各种各样的原因去鼓励作用域为基础的隐藏。这个设计来自于软件设计原则"最小特权原则"[^note-leastprivilege]，有时候也被叫做"最小权限"或者"最小暴露"。这个原则是软件设计的通常说法，用于模块/对象的API，你只能暴露需要的最小东西，然后将其他的都"隐藏"起来。

This principle extends to the choice of which scope to contain variables and functions. If all variables and functions were in the global scope, they would of course be accessible to any nested scope. But this would violate the "Least..." principle in that you are (likely) exposing many variables or functions which you should otherwise keep private, as proper use of the code would discourage access to those variables/functions.

这个原则延伸到选择哪个作用域来包含变量和方法。如果所有变量和方法在全局作用域中,他们当然可以被任何嵌套的作用域访问。但是这违反了"最小..."原则因为你会(大概)暴露多个你需要保持私有的变量或者方法，因为正确使用这些代码的方式是阻拦访问这些变量/方法。

For example:

举个例子:

```js
function doSomething(a) {
	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

function doSomethingElse(a) {
	return a - 1;
}

var b;

doSomething( 2 ); // 15
```

In this snippet, the `b` variable and the `doSomethingElse(..)` function are likely "private" details of how `doSomething(..)` does its job. Giving the enclosing scope "access" to `b` and `doSomethingElse(..)` is not only unnecessary but also possibly "dangerous", in that they may be used in unexpected ways, intentionally or not, and this may violate pre-condition assumptions of `doSomething(..)`.

在这段片段里,变量`b`和方法`doSomethingElse(..)`就好像`doSomething(..)`的"私有"细节。给包围他的作用域“存取” `b`和`doSomethingElse(..)`不仅仅是多余的而且可能是"危险"的,可能通过一些出乎意料的方式，故意的或者又不是,这可能违反了 `doSomething(..)`提前设定的责任。

A more "proper" design would hide these private details inside the scope of `doSomething(..)`, such as:

一个更"恰当"的设计会是隐藏`doSomething(..)`作用域的这些私有细节,类似：

```js
function doSomething(a) {
	function doSomethingElse(a) {
		return a - 1;
	}

	var b;

	b = a + doSomethingElse( a * 2 );

	console.log( b * 3 );
}

doSomething( 2 ); // 15
```

Now, `b` and `doSomethingElse(..)` are not accessible to any outside influence, instead controlled only by `doSomething(..)`. The functionality and end-result has not been affected, but the design keeps private details private, which is usually considered better software.

现在,`b` 和 `doSomethingElse(..)`将不受任何外部的影响不可读取,相反的只受`doSomething(..)`的控制。方法和最后的结果将不被受影响,设计为保持私有细节为私有,这通常被认为是更好的软件设计。

### Collision Avoidance

### 避免冲突

Another benefit of "hiding" variables and functions inside a scope is to avoid unintended collision between two different identifiers with the same name but different intended usages. Collision results often in unexpected overwriting of values.

把变量和方法放在作用域里隐藏的另外一个好处是避免两个不同的标识符有相同的名字但是不同的用法的无意间的冲突。冲突结果经常是出乎意料的覆盖重写了值。

For example:

举个例子：

```js
function foo() {
	function bar(a) {
		i = 3; // changing the `i` in the enclosing scope's for-loop
		console.log( a + i );
	}

	for (var i=0; i<10; i++) {
		bar( i * 2 ); // oops, infinite loop ahead!
	}
}

foo();
```

The `i = 3` assignment inside of `bar(..)` overwrites, unexpectedly, the `i` that was declared in `foo(..)` at the for-loop. In this case, it will result in an infinite loop, because `i` is set to a fixed value of `3` and that will forever remain `< 10`.

`i`在`foo(..)`中被声明并且循环,出乎意料的,`bar(..)`内部的`i = 3`赋值操作将他覆盖重写了。在这种情况下，结果将会是一个无线循环，因为`i`被设置成一个固定的值`3`结果将永远小于`< 10`。

The assignment inside `bar(..)` needs to declare a local variable to use, regardless of what identifier name is chosen. `var i = 3;` would fix the problem (and would create the previously mentioned "shadowed variable" declaration for `i`). An *additional*, not alternate, option is to pick another identifier name entirely, such as `var j = 3;`. But your software design may naturally call for the same identifier name, so utilizing scope to "hide" your inner declaration is your best/only option in that case.

`bar(..)`内部的赋值操作需要声明一个本地的变量来使用,不论选择什么名字的标识符。`var i = 3;`会修复这个问题(将会创建于一个之前提到过的为i声明"遮蔽变量")。此外,一个非代替的选项就是取完全不同的另外一个标识符，类似`var j = 3;`。但是你的软件原本的设计可能就是相同的标识符名称,所以这种情况下利用作用域"隐藏"内部的声明是你最好的选择。

#### Global "Namespaces"

#### 全局"命名空间"

A particularly strong example of (likely) variable collision occurs in the global scope. Multiple libraries loaded into your program can quite easily collide with each other if they don't properly hide their internal/private functions and variables.

变量冲突发生在全局作用域中一个特别常见的例子(基本上)。在你的程序中加载多个库如果他们没有更好的把他们的内部的/私有的方法和变量隐藏起来那么很容易就会发生冲突。

Such libraries typically will create a single variable declaration, often an object, with a sufficiently unique name, in the global scope. This object is then used as a "namespace" for that library, where all specific exposures of functionality are made as properties off that object (namespace), rather than as top-level lexically scoped identifiers themselves.

这类库通常会创建一个单独的变量声明，一般是一个对象，在全局作用域中用一个完全唯一的名称。这个对象作为库的一个"命名空间"被使用，所有的方法都作为对象(命名空间)的属性明确的暴露，而不是把自己作为最高层的词法作用标识符。

For example:

举例来说:

```js
var MyReallyCoolLibrary = {
	awesome: "stuff",
	doSomething: function() {
		// ...
	},
	doAnotherThing: function() {
		// ...
	}
};
```

#### Module Management

#### 模块管理

Another option for collision avoidance is the more modern "module" approach, using any of various dependency managers. Using these tools, no libraries ever add any identifiers to the global scope, but are instead required to have their identifier(s) be explicitly imported into another specific scope through usage of the dependency manager's various mechanisms.

避免冲突的另外一个选项是更现代化的"模块"方式，使用各种各样的任何一种依赖管理。使用这些工具，没有任何库会增加任何一个标识符到全局作用域中，但是相反的需要把他们的标识符明确的输入到特定的作用域中依赖管理器各种各样的机制来使用。

It should be observed that these tools do not possess "magic" functionality that is exempt from lexical scoping rules. They simply use the rules of scoping as explained here to enforce that no identifiers are injected into any shared scope, and are instead kept in private, non-collision-susceptible scopes, which prevents any accidental scope collisions.

应该可以看到这些工具不具有躲避词法作用域规则的"魔法"方法。他简单的使用这里讲解的作用域规则来强迫没有标识可以注入到任何共享的作用域中，相反的是保持私有的，无冲突不会受影响的作用域中，阻止任何偶然的作用域冲突创建。

As such, you can code defensively and achieve the same results as the dependency managers do without actually needing to use them, if you so choose. See the Chapter 5 for more information about the module pattern.

所以,如果你这样选择的话，你可以写防御性代码在不使用管理器的情况下达到同样的效果。见第五章获得更多模块模式的信息。

## Functions As Scopes

## 方法作用域

We've seen that we can take any snippet of code and wrap a function around it, and that effectively "hides" any enclosed variable or function declarations from the outside scope inside that function's inner scope.

我们知道我们可以任何一段代码然后用一个方法保住，然后有效的在作用域内部对外部的作用域"隐藏"任何被包围的变量或者方法声明。

For example:

举个例子:

```js
var a = 2;

function foo() { // <-- insert this

	var a = 3;
	console.log( a ); // 3

} // <-- and this
foo(); // <-- and this

console.log( a ); // 2
```

While this technique "works", it is not necessarily very ideal. There are a few problems it introduces. The first is that we have to declare a named-function `foo()`, which means that the identifier name `foo` itself "pollutes" the enclosing scope (global, in this case). We also have to explicitly call the function by name (`foo()`) so that the wrapped code actually executes.

尽管这个技巧可以"工作",这并非是个很好的注意。这样有几个问题需要介绍。首先我们声明了一个被取了名字的方法`foo()`，这意味着标识符名称`foo`"污染"了包围的作用域(全局,在这个例子里)。我们任然可以直接使用名称调用方法(`foo()`)所以实际上包裹的代码还是执行了。

It would be more ideal if the function didn't need a name (or, rather, the name didn't pollute the enclosing scope), and if the function could automatically be executed.

如果方法不需要名字(或者至少这个名字不会污染所属的作用域),而且这个方法可以自动被执行那我们就有更多的办法了。

Fortunately, JavaScript offers a solution to both problems.

幸运的是,JavaScript提供了解决全部问题的解决方案。

```js
var a = 2;

(function foo(){ // <-- insert this

	var a = 3;
	console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```

Let's break down what's happening here.

让我们拆解下这里发生了什么。

First, notice that the wrapping function statement starts with `(function...` as opposed to just `function...`. While this may seem like a minor detail, it's actually a major change. Instead of treating the function as a standard declaration, the function is treated as a function-expression.

首先,注意到一个包裹的方法声明以`(function...`开始代替`function...`。尽管这好像是微小的细节，事实上这是最主要的改变。方法被对待为一个方法表达式取代对待方法为一个标准的声明。

**Note:** The easiest way to distinguish declaration vs. expression is the position of the word "function" in the statement (not just a line, but a distinct statement). If "function" is the very first thing in the statement, then it's a function declaration. Otherwise, it's a function expression.

**注意:** 最简单区分声明和表达式方式是单词"function"在声明的位置(不仅仅是一行,而是明确的表达式)。如果声明在最开始的地方出现"function",那么就是方法声明,否则，就是方法表达式。

The key difference we can observe here between a function declaration and a function expression relates to where its name is bound as an identifier.

我们可以观察到方法声明和方法表达式最关键的区别是方法名称和识别符的绑定。

Compare the previous two snippets. In the first snippet, the name `foo` is bound in the enclosing scope, and we call it directly with `foo()`. In the second snippet, the name `foo` is not bound in the enclosing scope, but instead is bound only inside of its own function.

比较之前的两个代码片段。在第一个片段里，名称`foo`和包围他的作用域绑定，我们可以直接通过`foo()`来调用。在第二个片段，名称`foo`没有和包围的作用域绑定，但是相反的仅仅和他自己的方法绑定。

In other words, `(function foo(){ .. })` as an expression means the identifier `foo` is found *only* in the scope where the `..` indicates, not in the outer scope. Hiding the name `foo` inside itself means it does not pollute the enclosing scope unnecessarily.

换句话来说，`(function foo(){ .. })`作为一个表达式意味着标识符`foo`*只能* 在`..`显示的作用域里找到，不能再外部的作用域中找到。将名称`foo`隐藏在自己内部意味着不会在不必要的情况下污染包围他的作用域。

### Anonymous vs. Named

### 匿名 vs. 取名

You are probably most familiar with function expressions as callback parameters, such as:

你可能已经很熟悉方法表达式作为回调的参数,类似：

```js
setTimeout( function(){
	console.log("I waited 1 second!");
}, 1000 );
```

This is called an "anonymous function expression", because `function()...` has no name identifier on it. Function expressions can be anonymous, but function declarations cannot omit the name -- that would be illegal JS grammar.

这被叫做一个"匿名方法表达式",因为`function()...`没有名称标识符。方法表达式可以是匿名的，但是方法声明不可以遗漏名称 -- 那将是不合法的JS语法。

Anonymous function expressions are quick and easy to type, and many libraries and tools tend to encourage this idiomatic style of code. However, they have several draw-backs to consider:

匿名方法表达式可以快速容易的编写，很多的库和方法倾向于鼓励这种理想风格的代码。然而，这里也有几个缺点需要考虑:

1. Anonymous functions have no useful name to display in stack traces, which can make debugging more difficult.

1. 匿名方法没有有用的名称在堆栈跟踪里显示，这让调试变得更加困难。

2. Without a name, if the function needs to refer to itself, for recursion, etc., the **deprecated** `arguments.callee` reference is unfortunately required. Another example of needing to self-reference is when an event handler function wants to unbind itself after it fires.

2. 没有了名字,如果方法需要引用他自己，来重新执行，等等，那不幸的需要使用**不推荐的**`arguments.callee`来引用。另外需要自己引用的例子是当一个事件处理者需要在触发后解绑他的自己的情况。

3. Anonymous functions omit a name that is often helpful in providing more readable/understandable code. A descriptive name helps self-document the code in question.

3. 匿名方法省略了名字而名字常常有助于带来更多的可读性/可理解性代码。一个描述的名字在有问题的时候可以帮助作为代码自己的文档。

**Inline function expressions** are powerful and useful -- the question of anonymous vs. named doesn't detract from that. Providing a name for your function expression quite effectively addresses all these draw-backs, but has no tangible downsides. The best practice is to always name your function expressions:

**内联方法表达式** 是很强大和有用的 -- 匿名 vs. 取名问题没有贬低他。给你方法提供一个名字会非常有效的解决所有的缺点,而且没有实质上的缺点。最佳的实践是永远给你的方法表达式取名：

```js
setTimeout( function timeoutHandler(){ // <-- Look, I have a name!
	console.log( "I waited 1 second!" );
}, 1000 );
```

### Invoking Function Expressions Immediately

### 立刻执行方法表达式

```js
var a = 2;

(function foo(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

Now that we have a function as an expression by virtue of wrapping it in a `( )` pair, we can execute that function by adding another `()` on the end, like `(function foo(){ .. })()`. The first enclosing `( )` pair makes the function an expression, and the second `()` executes the function.

现在我们有了一个方法作为表达式凭借一对`( )`包裹，我们可以在最后增加另外一对`()`来调用，就像`(function foo(){ .. })()`。第一对包括的`( )`让方法变成一个表达式，然后第二对`()`执行了这个方法。

This pattern is so common, a few years ago the community agreed on a term for it: **IIFE**, which stands for **I**mmediately **I**nvoked **F**unction **E**xpression.

这个模式是很常见的，几年之前社区就为这个模式达成了一个术语叫做： **IIFE**，他就代表**I**mmediately **I**nvoked **F**unction **E**xpression。

Of course, IIFE's don't need names, necessarily -- the most common form of IIFE is to use an anonymous function expression. While certainly less common, naming an IIFE has all the aforementioned benefits over anonymous function expressions, so it's a good practice to adopt.

当然,IIFE不需要名称，名字不是必须的 -- 最常见的IIFE是使用匿名的方法表达式。虽然确实比常见的少，但有名字的IIFE相比匿名表达式拥有所有上述提到过的益处，所以采用这个是一个好的实践。

```js
var a = 2;

(function IIFE(){

	var a = 3;
	console.log( a ); // 3

})();

console.log( a ); // 2
```

There's a slight variation on the traditional IIFE form, which some prefer: `(function(){ .. }())`. Look closely to see the difference. In the first form, the function expression is wrapped in `( )`, and then the invoking `()` pair is on the outside right after it. In the second form, the invoking `()` pair is moved to the inside of the outer `( )` wrapping pair.

这是在传统的IIFE上稍加变化，虽然有些人偏好`(function(){ .. }())`。仔细观察可以看出不同。在第一个形式中，方法表达式被`( )`包围，然后用一对放在外部在右边紧随其后的`()`触发。在第二个形式中，用于触发的一对`()`被移动到了外部用于包围的一对`( )`里面。

These two forms are identical in functionality. **It's purely a stylistic choice which you prefer.**

这两种形式在方法中完全相同。**这完全是一个代码风格的选择取决你更喜欢哪个**。

Another variation on IIFE's which is quite common is to use the fact that they are, in fact, just function calls, and pass in argument(s).

另外一种非常常见的IIFE的变化是使用最真实的自己，事实上，只是调用方法，然后将参数传进去。

For instance:

举例来说：

```js
var a = 2;

(function IIFE( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```

We pass in the `window` object reference, but we name the parameter `global`, so that we have a clear stylistic delineation for global vs. non-global references. Of course, you can pass in anything from an enclosing scope you want, and you can name the parameter(s) anything that suits you. This is mostly just stylistic choice.

我们传入一个`window`对象的引用，但是我们给参数取名为`global`，这样我们对全局vs.非全局引用有了一个清楚的文体上的的解释。当然，你可以传入任何你想要的来自包围你的作用域的东西，然后你可以给参数取任何你觉得合适的名字。这大多情况下只是代码风格的选择。

Another application of this pattern addresses the (minor niche) concern that the default `undefined` identifier might have its value incorrectly overwritten, causing unexpected results. By naming a parameter `undefined`, but not passing any value for that argument, we can guarantee that the `undefined` identifier is in fact the undefined value in a block of code:

这个模式另外一种应用是设法解决(微小的问题)考虑到默认的`undefined`标识符可能会被不正确的值重写，造成出乎意料的结果。给参数取名为`undefined`，但是不传入任何值给参数，这样我们可以保证`undefined`标识符在这段代码的内部是undefined本身的值。

```js
undefined = true; // setting a land-mine for other code! avoid!

(function IIFE( undefined ){

	var a;
	if (a === undefined) {
		console.log( "Undefined is safe here!" );
	}

})();
```

Still another variation of the IIFE inverts the order of things, where the function to execute is given second, *after* the invocation and parameters to pass to it. This pattern is used in the UMD (Universal Module Definition) project. Some people find it a little cleaner to understand, though it is slightly more verbose.

任然是IIFE的另外一种变化是颠倒事情的顺序，将方法调用放在第二步，在调用和将参数传入*之后*。这个模式在UMD(Universal Module Definition)项目中被使用。有些人发现这会更清楚被理解，虽然这有些轻微的冗长。

```js
var a = 2;

(function IIFE( def ){
	def( window );
})(function def( global ){

	var a = 3;
	console.log( a ); // 3
	console.log( global.a ); // 2

});
```

The `def` function expression is defined in the second-half of the snippet, and then passed as a parameter (also called `def`) to the `IIFE` function defined in the first half of the snippet. Finally, the parameter `def` (the function) is invoked, passing `window` in as the `global` parameter.

`def`方法表达式被定义在代码片段的第二部分，然后被作为参数(仍然叫做`def`)传入被定义在代码片段第一部分的`IIFE`方法。最后，参数`def`(那个方法)被调用，传入`window`作为`global`参数。

## Blocks As Scopes

## 块作用域

While functions are the most common unit of scope, and certainly the most wide-spread of the design approaches in the majority of JS in circulation, other units of scope are possible, and the usage of these other scope units can lead to even better, cleaner to maintain code.

尽管方法是最常见作用域单元，无疑是在大多数JS流传最广的设计中传播最广的，但是其他作用域的单位也是合理的，使用这些其他单位的作用域可以更好，更清楚的维护代码。

Many languages other than JavaScript support Block Scope, and so developers from those languages are accustomed to the mindset, whereas those who've primarily only worked in JavaScript may find the concept slightly foreign.

许多其他的语言相比JavaScript支持块作用域，所以这些语言的开发者习惯于这个观点，然而对那些主要使用JavaScript工作的人来说这个观点有些陌生。

But even if you've never written a single line of code in block-scoped fashion, you are still probably familiar with this extremely common idiom in JavaScript:

但是即使你没有写过一行块作用域风格的代码，你仍然可能熟悉这个极度常见JavaScript语法:

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```

We declare the variable `i` directly inside the for-loop head, most likely because our *intent* is to use `i` only within the context of that for-loop, and essentially ignore the fact that the variable actually scopes itself to the enclosing scope (function or global).

我们直接在for循环的头部声明了变量`i`，很像块作用域的写法因为我们的*目的*是只在for循环的内部使用`i`，然后本质上忽略了一个事实变量实际上属于包围他的作用域(方法或者全局作用域)。

That's what block-scoping is all about. Declaring variables as close as possible, as local as possible, to where they will be used. Another example:

这就是关于块级作用域。声明变量对于他要使用的地方越封闭越好，越本地越好(译者注:变量只属于使用他的块作用域)。另外一个例子

```js
var foo = true;

if (foo) {
	var bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}
```

We are using a `bar` variable only in the context of the if-statement, so it makes a kind of sense that we would declare it inside the if-block. However, where we declare variables is not relevant when using `var`, because they will always belong to the enclosing scope. This snippet is essentially "fake" block-scoping, for stylistic reasons, and relying on self-enforcement not to accidentally use `bar` in another place in that scope.

我们只在if表达式的块内部使用变量`bar`，所以这有点我们只在if块的内部声明他的意思。然而，我们在哪里声明变量并不取决于我们什么时候使用`var`，因为变量始终属于包围他的作用域。这个片段本质上是"伪装"成块作用域，因为代码风格的原因，假装强迫自己不能再意外的情况下在作用域的其他地方使用`bar`。

Block scope is a tool to extend the earlier "Principle of Least ~~Privilege~~ Exposure" [^note-leastprivilege] from hiding information in functions to hiding information in blocks of our code.

块作用域是更简单的扩展"最小原则 ~~特权~~ 暴露"[^note-leastprivilege]的工具，让我们的代码从隐藏信息到方法里到隐藏信息到块里。

Consider the for-loop example again:

再考虑一下for循环的例子：

```js
for (var i=0; i<10; i++) {
	console.log( i );
}
```

Why pollute the entire scope of a function with the `i` variable that is only going to be (or only *should be*, at least) used for the for-loop?

为什么变量`i`只会在(或者至少应该在)for循环中使用却要污染整个作用域？

But more importantly, developers may prefer to *check* themselves against accidentally (re)using variables outside of their intended purpose, such as being issued an error about an unknown variable if you try to use it in the wrong place. Block-scoping (if it were possible) for the `i` variable would make `i` available only for the for-loop, causing an error if `i` is accessed elsewhere in the function. This helps ensure variables are not re-used in confusing or hard-to-maintain ways.

但是更重要的，开发者最好去*检查*他自己以防在目的以外的地方无意的使用(再使用)变量，如果你在错误的地方使用变量会引起不知道变量的错误。在块作用域(如果可能的话)中变量`i`会使变量`i`只属于for循环，如果`i`在方法的其他地方被读取将引发错误。这有助于确保变量不会被不清楚的方式或者难以维护的方式被使用。

But, the sad reality is that, on the surface, JavaScript has no facility for block scope.

但是，悲伤的现实是，从表面上看，JavaScript没有块作用域的能力。(facility:a natural ability to learn or do sth easily)

That is, until you dig a little further.

至少目前来看是这样，除非你挖掘更深入一些。

### `with`

We learned about `with` in Chapter 2. While it is a frowned upon construct, it *is* an example of (a form of) block scope, in that the scope that is created from the object only exists for the lifetime of that `with` statement, and not in the enclosing scope.

我们在第二章已经学过`with`。尽管这是一个让人皱眉的概念，但是他*是*一种块作用域的例子(形式)，从对象创建的作用域和对象本身只存在于`with`表达式的生命周期，而不在包围他的作用域中。

### `try/catch`

It's a *very* little known fact that JavaScript in ES3 specified the variable declaration in the `catch` clause of a `try/catch` to be block-scoped to the `catch` block.

一个很少知道的事实是JavaScript在ES3提出在`try/catch`中的`catch`分句中是一个块级作用域声明在里面的变量只属于`catch`。

For instance:

事例:

```js
try {
	undefined(); // illegal operation to force an exception!
}
catch (err) {
	console.log( err ); // works!
}

console.log( err ); // ReferenceError: `err` not found
```

As you can see, `err` exists only in the `catch` clause, and throws an error when you try to reference it elsewhere.

就像你看到的,`err`只存在于`catch`分句里,当你在任何其他地方尝试引用他都将抛出一个错误。

**Note:** While this behavior has been specified and true of practically all standard JS environments (except perhaps old IE), many linters seem to still complain if you have two or more `catch` clauses in the same scope which each declare their error variable with the same identifier name. This is not actually a re-definition, since the variables are safely block-scoped, but the linters still seem to, annoyingly, complain about this fact.

**注意:** 虽然这个表现方式已经被明确规定而且在几乎所有的标准JS环境中都是正确的(也许除了老IE)，但是如果你在同一个作用域中有两个或者多个`catch`分句将错误变量用同样的标识符声明那么许多的linter还是回警告的。这事实上不是重复定义，因为每个变量在块级作用域中都是安全的，但是linters仍然把他们事实视作是错误的值得警告的。

To avoid these unnecessary warnings, some devs will name their `catch` variables `err1`, `err2`, etc. Other devs will simply turn off the linting check for duplicate variable names.

为了避免这些不必要的警告，一些开发者会将`catch`变量取名为`err1`, `err2`,等等。其他的开发者会简单的关闭linting的检查重复变量名称的开关。

The block-scoping nature of `catch` may seem like a useless academic fact, but see Appendix B for more information on just how useful it might be.

`catch`的原本的块级作用域似乎看起来是无用的学术事实，但是如何使他更有用的信息见附录B。

### `let`

Thus far, we've seen that JavaScript only has some strange niche behaviors which expose block scope functionality. If that were all we had, and *it was* for many, many years, then block scoping would not be terribly useful to the JavaScript developer.

因此，我们就可以看见JavaScript仅仅暴露一些奇怪举止的块级作用域方法。如果这就是我们拥有的关于块级作用域的全部，然而这确实是很多年*以来*我们所拥有的，那么这对JavaScript开发者也不会是非常有用的。

Fortunately, ES6 changes that, and introduces a new keyword `let` which sits alongside `var` as another way to declare variables.

幸运的是,ES6改变了这个，介绍了一个新的关键字`let`他和`var`一样但是作为另外一种声明变量的方式。

The `let` keyword attaches the variable declaration to the scope of whatever block (commonly a `{ .. }` pair) it's contained in. In other words, `let` implicitly hijacks any block's scope for its variable declaration.

`let`关键字将变量申明依附在任何他所在的块级作用域中(通常是一对`{ .. }`)。换句话来说，`let`隐形的劫持任意一个块级作用域给变量声明。

```js
var foo = true;

if (foo) {
	let bar = foo * 2;
	bar = something( bar );
	console.log( bar );
}

console.log( bar ); // ReferenceError
```

Using `let` to attach a variable to an existing block is somewhat implicit. It can confuse you if you're not paying close attention to which blocks have variables scoped to them, and are in the habit of moving blocks around, wrapping them in other blocks, etc., as you develop and evolve code.

使用`let`使变量依附在一个存在的块上有些隐晦。如果在你开发设计代码的时候你没有仔细注意到变量所在的哪个块是他的作用域而是习惯性的将他在块之间移动将他包含在其他块里那这会让你混乱，

Creating explicit blocks for block-scoping can address some of these concerns, making it more obvious where variables are attached and not. Usually, explicit code is preferable over implicit or subtle code. This explicit block-scoping style is easy to achieve, and fits more naturally with how block-scoping works in other languages:

给块作用域创建明确的块可以处理一些这类顾虑，使变量是否依附某个块作用域更明显。一般来说。明确的代码比不明确或者不明显的代码更好。明确的块级作用域风格更容易达到某个目的，这也让JavaScript的块级作用域更像自他语言的块级作用域如何工作一般自然。

```js
var foo = true;

if (foo) {
	{ // <-- explicit block
		let bar = foo * 2;
		bar = something( bar );
		console.log( bar );
	}
}

console.log( bar ); // ReferenceError
```

We can create an arbitrary block for `let` to bind to by simply including a `{ .. }` pair anywhere a statement is valid grammar. In this case, we've made an explicit block *inside* the if-statement, which may be easier as a whole block to move around later in refactoring, without affecting the position and semantics of the enclosing if-statement.

我们可以通过简单的写一对`{ .. }`来为`let`绑定创建任意块将表达式包含在里面这都是合理的语法。在这个情况里，我们在if表达式*里*创建一个明确的块，这也许让整个块移动和以后的重构更容易，也不会影响位置和if表达式包围块的意义。

**Note:** For another way to express explicit block scopes, see Appendix B.

**注意:** 表达明确块作用域的其他方式，见附录B。

In Chapter 4, we will address hoisting, which talks about declarations being taken as existing for the entire scope in which they occur.

在第四章中，我们会介绍提升，谈论关于声明会在他执行的整个作用域中存在。

However, declarations made with `let` will *not* hoist to the entire scope of the block they appear in. Such declarations will not observably "exist" in the block until the declaration statement.

然而，`let`声明将*不会*在在他执行的整个作用域中提升。这个声明将不会观察到"存在"于块中直到声明表达式执行。

```js
{
   console.log( bar ); // ReferenceError!
   let bar = 2;
}
```

#### Garbage Collection

#### 垃圾回收(GC)

Another reason block-scoping is useful relates to closures and garbage collection to reclaim memory. We'll briefly illustrate here, but the closure mechanism is explained in detail in Chapter 5.

另外一个块作用域有用的原因是终止和执行垃圾回收来取回内存。我们会在这里简单的距离说明，但是终止机制的细节会在第五章解释。

Consider:

考虑一下：

```js
function process(data) {
	// do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

The `click` function click handler callback doesn't *need* the `someReallyBigData` variable at all. That means, theoretically, after `process(..)` runs, the big memory-heavy data structure could be garbage collected. However, it's quite likely (though implementation dependent) that the JS engine will still have to keep the structure around, since the `click` function has a closure over the entire scope.

`click`方法的click处理回调完全不*需要*`someReallyBigData`变量。这意味着，理论上来说，在`process(..)`执行后，有大量的内存数据可以被垃圾回收。然而，很有可能(取决于整个实现)JS引擎将会任然保持这个结构，直到`click`方法整个作用域终止。

Block-scoping can address this concern, making it clearer to the engine that it does not need to keep `someReallyBigData` around:

块级作用域可以设法解决这个顾虑，是引擎不在需要保持`someReallyBigData`让他更干净：

```js
function process(data) {
	// do something interesting
}

// anything declared inside this block can go away after!
{
	let someReallyBigData = { .. };

	process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
	console.log("button clicked");
}, /*capturingPhase=*/false );
```

Declaring explicit blocks for variables to locally bind to is a powerful tool that you can add to your code toolbox.

为变量局部绑定声明明确的块是一个很强大的工具你可以加入到你的代码工具箱里。

#### `let` Loops

#### `let` 循环

A particular case where `let` shines is in the for-loop case as we discussed previously.

`let`可以发光的一个特别的例子是我们之前讨论过的for循环。

```js
for (let i=0; i<10; i++) {
	console.log( i );
}

console.log( i ); // ReferenceError
```

Not only does `let` in the for-loop header bind the `i` to the for-loop body, but in fact, it **re-binds it** to each *iteration* of the loop, making sure to re-assign it the value from the end of the previous loop iteration.

不仅仅是在for循环的头部`let`把`i`绑定在for循环的正文，事实上，每*循环*一次都**重新绑定他**，确保重新复制的值是上一次循环最后的值。

Here's another way of illustrating the per-iteration binding behavior that occurs:

这里是另外一种方式来举例说明执行前一次绑定行为：

```js
{
	let j;
	for (j=0; j<10; j++) {
		let i = j; // re-bound for each iteration!
		console.log( i );
	}
}
```

The reason why this per-iteration binding is interesting will become clear in Chapter 5 when we discuss closures.

前一次循环绑定为什么有趣在我们第五章讨论闭包的时候就清楚了。

Because `let` declarations attach to arbitrary blocks rather than to the enclosing function's scope (or global), there can be gotchas where existing code has a hidden reliance on function-scoped `var` declarations, and replacing the `var` with `let` may require additional care when refactoring code.

因为`let`声明依附于任意一个块而不是包含他的方法作用于(或者全局作用域)，所以在重构代码时将`var`替换成`let`时，需要抓住隐藏依赖于`var`声明的作用域上的已经存在的代码，需要额外的关心。

Consider:

考虑一下：

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	if (baz > bar) {
		console.log( baz );
	}

	// ...
}
```

This code is fairly easily re-factored as:

这个代码相当简单的重构为：

```js
var foo = true, baz = 10;

if (foo) {
	var bar = 3;

	// ...
}

if (baz > bar) {
	console.log( baz );
}
```

But, be careful of such changes when using block-scoped variables:

但是，当使用块级作用域改变这种情况是要小心：

```js
var foo = true, baz = 10;

if (foo) {
	let bar = 3;

	if (baz > bar) { // <-- don't forget `bar` when moving!
		console.log( baz );
	}
}
```

See Appendix B for an alternate (more explicit) style of block-scoping which may provide easier to maintain/refactor code that's more robust to these scenarios.

附录B介绍了一种块作用域的（更明确）替代形式它可能会提供在这些场景下更易于维护/重构的更健壮的代码。

### `const`

In addition to `let`, ES6 introduces `const`, which also creates a block-scoped variable, but whose value is fixed (constant). Any attempt to change that value at a later time results in an error.

除了`let`之外，ES6还介绍了`const`，虽然他也创建一个块级作用域的变量，但是他的值是固定的(常数)。任何在晚些时候试图修改他值的结果都是一个错误。

```js
var foo = true;

if (foo) {
	var a = 2;
	const b = 3; // block-scoped to the containing `if`

	a = 3; // just fine!
	b = 4; // error!
}

console.log( a ); // 3
console.log( b ); // ReferenceError!
```

## Review (TL;DR)

## 回顾 (TL;DR)

Functions are the most common unit of scope in JavaScript. Variables and functions that are declared inside another function are essentially "hidden" from any of the enclosing "scopes", which is an intentional design principle of good software.

方法在JavaScript中是最常见的作用域单位。在另外一个方法中声明的变量和方法本质上"隐藏"于任何外部的"作用域"，这是传统设计观念中更好的软件。

But functions are by no means the only unit of scope. Block-scope refers to the idea that variables and functions can belong to an arbitrary block (generally, any `{ .. }` pair) of code, rather than only to the enclosing function.

但是方法并不是唯一的作用于单位。块级作用域提到变量和方法可以属于任意一个块(通常,任意一对`{ .. }`)的代码而不是仅仅属于包围他的方法。

Starting with ES3, the `try/catch` structure has block-scope in the `catch` clause.

从ES3开始，`try/catch`结构中的`catch`分句是块级作用域。

In ES6, the `let` keyword (a cousin to the `var` keyword) is introduced to allow declarations of variables in any arbitrary block of code. `if (..) { let a = 2; }` will declare a variable `a` that essentially hijacks the scope of the `if`'s `{ .. }` block and attaches itself there.

在ES6中，介绍了`let`关键字(`var`关键字的表兄)它允许被声明的变量属于任意一个代码块，`if (..) { let a = 2; }`声明了变量`a` 他劫持了`if`的`{ .. }`块作用域然后将自己依附于这个块作用域上。

Though some seem to believe so, block scope should not be taken as an outright replacement of `var` function scope. Both functionalities co-exist, and developers can and should use both function-scope and block-scope techniques where respectively appropriate to produce better, more readable/maintainable code.

虽然有些人似乎这么认为，但块作用域应该不能被认为是用来完全替代`var`方法作用域的。两个方法共存，开发者可以和应该同时使用方法作用域和块作用域技巧来分别适当的使用让程序更好，使代码更可读/可维护。

[^note-leastprivilege]: [Principle of Least Privilege](http://en.wikipedia.org/wiki/Principle_of_least_privilege)

