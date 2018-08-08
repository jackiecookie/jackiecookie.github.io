---
layout: post
title: 'You Don''t Know JS: Scope & Closures-Chapter 2: Lexical Scope'
date: 2017-07-29 12:15:45
tags:
---

# You Don't Know JS: Scope & Closures

# 你不知道的JS：作用域和闭包
# Chapter 2: Lexical Scope

#第二章：词法作用域

In Chapter 1, we defined "scope" as the set of rules that govern how the *Engine* can look up a variable by its identifier name and find it, either in the current *Scope*, or in any of the *Nested Scopes* it's contained within.

在第一章,我们把"作用域"定义为设置一个规则管理引擎如何根据定义好的变量名称找到他，不论是在当前的作用域,或者是任何一个他被包含着的嵌套作用域里。

There are two predominant models for how scope works. The first of these is by far the most common, used by the vast majority of programming languages. It's called **Lexical Scope**, and we will examine it in-depth. The other model, which is still used by some languages (such as Bash scripting, some modes in Perl, etc.) is called **Dynamic Scope**.

作用域如何工作主要有两个模型。第一个是目前为止最常见的,在大多数的编程语言中被使用。叫做**词法作用域**,然后我们接下来会更深入的调查。另外一种模型,在一些语言中仍然被使用(像Bash脚本,Perl中的一些模式,等等)叫做**动态作用域**。

Dynamic Scope is covered in Appendix A. I mention it here only to provide a contrast with Lexical Scope, which is the scope model that JavaScript employs.

动态作用域在附录A中介绍。我在这里提到它只是和词法作用域做一个对照，而词法作用域这种模型也是JavaScript使用的。

## Lex-time

## 词法分析时间

As we discussed in Chapter 1, the first traditional phase of a standard language compiler is called lexing (aka, tokenizing). If you recall, the lexing process examines a string of source code characters and assigns semantic meaning to the tokens as a result of some stateful parsing.

就像我们在第一章里讨论过的,标准的语言编译器在第一个传统阶段叫做词法分析(又叫做分词)。如果你回想一下，词法分析的过程就是检查源代码字符组成的字符串然后根据词法的意义分词作为有状态的解析结果。

It is this concept which provides the foundation to understand what lexical scope is and where the name comes from.

这个观点提供了了解什么是词法作用域和这个名字是从哪里来的基础。

To define it somewhat circularly, lexical scope is scope that is defined at lexing time. In other words, lexical scope is based on where variables and blocks of scope are authored, by you, at write time, and thus is (mostly) set in stone by the time the lexer processes your code.

要定义它某种程度上有点绕，词法作用域是在词法分析时间定义的作用域。换句话来说，词法作用域是取决于你在写的时候以变量和作用域的块在哪里编写为依据，因此在分词器处理你代码的时候(基本上)是不变的。

**Note:** We will see in a little bit there are some ways to cheat lexical scope, thereby modifying it after the lexer has passed by, but these are frowned upon. It is considered best practice to treat lexical scope as, in fact, lexical-only, and thus entirely author-time in nature.

**注意:** 我们会看到有些方法他们可以骗过词法作用域,借由此在词法分析器传进来之前修改他，但是这会显得很迷惑。事实上，被认为是对待作用域最佳的实践是最好是在词法分析时间,也就是原始的整个编写的时间。

Let's consider this block of code:

让我们来考虑一下这段代码：

```js
function foo(a) {

	var b = a * 2;

	function bar(c) {
		console.log( a, b, c );
	}

	bar(b * 3);
}

foo( 2 ); // 2 4 12
```

There are three nested scopes inherent in this code example. It may be helpful to think about these scopes as bubbles inside of each other.

这里是三个被嵌套的固有的(inherent:that is a basic or permanent part of sb/sth and that cannot be removed 在这里我翻译做固有的结合前面的set in stone)作用域在这个代码例子中。把这些作用域想象成气泡存在于其他气泡中可能会对理解有帮助。

<img src="fig2.png" width="500">

**Bubble 1** encompasses the global scope, and has just one identifier in it: `foo`.

**气泡 1** 包括在全局作用域,只有一个标识符在里面: `foo`。

**Bubble 2** encompasses the scope of `foo`, which includes the three identifiers: `a`, `bar` and `b`.

**气泡 2** 包括作用域`foo`,包含了3个标识符: `a`, `bar` and `b`。

**Bubble 3** encompasses the scope of `bar`, and it includes just one identifier: `c`.

**气泡 3** 包括作用域`bar`,它只包含了一个标识符: `c`。

Scope bubbles are defined by where the blocks of scope are written, which one is nested inside the other, etc. In the next chapter, we'll discuss different units of scope, but for now, let's just assume that each function creates a new bubble of scope.

作用域气泡是被定义在作用域区块书写的地方,一个嵌套一个，等等。在下一个章节中，我们会讨论不同的作用域单位，但是现在，我们只假设每个方法创建一个新的作用域气泡。

The bubble for `bar` is entirely contained within the bubble for `foo`, because (and only because) that's where we chose to define the function `bar`.

`bar`气泡被`foo`气泡完全的包含，因为(而且只因为)我们选择在哪里定义方法`bar`。

Notice that these nested bubbles are strictly nested. We're not talking about Venn diagrams where the bubbles can cross boundaries. In other words, no bubble for some function can simultaneously exist (partially) inside two other outer scope bubbles, just as no function can partially be inside each of two parent functions.

注意这里的嵌套气泡是严格嵌套的。我们讨论的不是气泡可以穿过边界的文氏图(Venn diagrams)。换句话来说,方法的气泡不能同时存在(部分)于两个其他的外层气泡中，就像没有方法可以部分存在于另外两个父级方法中。

### Look-ups

The structure and relative placement of these scope bubbles fully explains to the *Engine* all the places it needs to look to find an identifier.

这些作用域气泡的结构和对应的位置可以完全解释引擎查找标识符需要查找的所有地方。

In the above code snippet, the *Engine* executes the `console.log(..)` statement and goes looking for the three referenced variables `a`, `b`, and `c`. It first starts with the innermost scope bubble, the scope of the `bar(..)` function. It won't find `a` there, so it goes up one level, out to the next nearest scope bubble, the scope of `foo(..)`. It finds `a` there, and so it uses that `a`. Same thing for `b`. But `c`, it does find inside of `bar(..)`.

在上面的代码片段中，引擎执行`console.log(..)`表达式然后去查看三个引用变量`a`, `b`, 和 `c`。首先从最内层的作用域气泡开始，也就是`bar(..)`方法的作用域。在那里找不到`a`，所以它往上一层，来到上一个最近的作用域气泡，`foo(..)`的作用域。在那里找到了 `a`，然后就使用那个 `a`。查找`b`做了同样的事情。但是 `c`，在`bar(..)`中可以找到。

Had there been a `c` both inside of `bar(..)` and inside of `foo(..)`, the `console.log(..)` statement would have found and used the one in `bar(..)`, never getting to the one in `foo(..)`.

假设`c`同时存在于`bar(..)`和`foo(..)`中，`console.log(..)`表达式会找到和使用`bar(..)`中的一个,永远不会得到`foo(..)`中的。

**Scope look-up stops once it finds the first match**. The same identifier name can be specified at multiple layers of nested scope, which is called "shadowing" (the inner identifier "shadows" the outer identifier). Regardless of shadowing, scope look-up always starts at the innermost scope being executed at the time, and works its way outward/upward until the first match, and stops.

**作用域查找一旦找到第一个匹配的就会停止**。相同名字的标识符可以在嵌套作用域中的多个层中被声明，这个叫做"阴影"(内层的标识符会遮住外层的标识符)。不管有没有遮住(Regardless of:paying not attention;treating sth as not being important)，在执行的时候作用域总是从最内层的作用域开始查找,然后一直往外层/上层找知道第一次找到，然后停止

**Note:** Global variables are also automatically properties of the global object (`window` in browsers, etc.), so it *is* possible to reference a global variable not directly by its lexical name, but instead indirectly as a property reference of the global object.

**注意:** 全局变量始终是全局对象(浏览器中的`window`,等等。)的自动属性，所以不直接使用他的词法名字通过引用全局变量的属性直接饮用全局变量是可能的，

```js
window.a
```

This technique gives access to a global variable which would otherwise be inaccessible due to it being shadowed. However, non-global shadowed variables cannot be accessed.

这个技巧使那些因为被遮蔽而无法访问的全局变量有了访问的途径。然而,非全局被遮蔽的变量的变量是不能访问的。

No matter *where* a function is invoked from, or even *how* it is invoked, its lexical scope is **only** defined by where the function was declared.

不管方法在 *哪里* 触发,或者甚至 *如何* 触发,他的词法作用域**只**跟方法在哪里被声明有关。

The lexical scope look-up process *only* applies to first-class identifiers, such as the `a`, `b`, and `c`. If you had a reference to `foo.bar.baz` in a piece of code, the lexical scope look-up would apply to finding the `foo` identifier, but once it locates that variable, object property-access rules take over to resolve the `bar` and `baz` properties, respectively.

词法作用查找过程*只*会查找一级分类标识符,像`a`, `b`, 和 `c`。如果有对`foo.bar.baz`这小段代码的引用，词法作用域会申请查找`foo`标识符，但是一旦找出这个变量，对象属性访问规则会分别解析 `bar`和`baz`属性。

## Cheating Lexical

## 欺骗词法

If lexical scope is defined only by where a function is declared, which is entirely an author-time decision, how could there possibly be a way to "modify" (aka, cheat) lexical scope at run-time?

如果词法作用域的定义只跟方法在哪里声明有关，完全是书写的时候决定，那么如何在运行时间"修改"(欺骗)词法作用域？

JavaScript has two such mechanisms. Both of them are equally frowned-upon in the wider community as bad practices to use in your code. But the typical arguments against them are often missing the most important point: **cheating lexical scope leads to poorer performance.**

JavaScript有两种这样的机制。在你的代码中使用任何一种机制在大部分的社区中都被认为是不好的实践而且是不被推荐的。但是和有代表性的争论相反的是他们经常忽略了最重要的点:**欺骗词法作用域会带来更差的性能。**

Before I explain the performance issue, though, let's look at how these two mechanisms work.

在我解释这个性能问题之前,让我们来看一下这两种机制是如何工作的。

### `eval`

### `eval`

The `eval(..)` function in JavaScript takes a string as an argument, and treats the contents of the string as if it had actually been authored code at that point in the program. In other words, you can programmatically generate code inside of your authored code, and run the generated code as if it had been there at author time.

`eval(..)`方法在JavaScript中需要一个字符串(string)作为参数，然后把字符串的内容当做真正书写在那个地方的代码一样对待。换句话来说，你可以按计划生成代码在你已经写好的代码里，然后执行生成代码好像他书写的时候就在那一样。

Evaluating `eval(..)` (pun intended) in that light, it should be clear how `eval(..)` allows you to modify the lexical scope environment by cheating and pretending that author-time (aka, lexical) code was there all along.

把`eval(..)`评估成这样，必须要清楚`eval(..)`如何允许你用欺骗的方式去修改词法作用域环境然后假装代码在书写时间(词法分析)就已经在那边很长时间了。

On subsequent lines of code after an `eval(..)` has executed, the *Engine* will not "know" or "care" that the previous code in question was dynamically interpreted and thus modified the lexical scope environment. The *Engine* will simply perform its lexical scope look-ups as it always does.

一个`eval(..)`执行后的按照顺序的代码,引擎会不"知道"或者不"关心"它之前的代码是否是动态解释的然后修改词法作用域环境。引擎仅仅使用词法作用域查看就像他始终干的一样。

Consider the following code:

考虑一下下面的代码：

```js
function foo(str, a) {
	eval( str ); // cheating!
	console.log( a, b );
}

var b = 2;

foo( "var b = 3;", 1 ); // 1 3
```

The string `"var b = 3;"` is treated, at the point of the `eval(..)` call, as code that was there all along. Because that code happens to declare a new variable `b`, it modifies the existing lexical scope of `foo(..)`. In fact, as mentioned above, this code actually creates variable `b` inside of `foo(..)` that shadows the `b` that was declared in the outer (global) scope.

这里的字符串 `"var b = 3;"`在`eval(..)`调用的位置，当成是已经在那里很久的代码被处理。因为这个代码声明了一个新的变量 `b`，他修改已经存在的词法作用域`foo(..)`。事实上，我们上面提到的，这个代码事实上在 `foo(..)`里面新创建了变量 `b`遮住了在外层(全局)作用域中被声明的`b`。

When the `console.log(..)` call occurs, it finds both `a` and `b` in the scope of `foo(..)`, and never finds the outer `b`. Thus, we print out "1 3" instead of "1 2" as would have normally been the case.

当`console.log(..)`调用执行，他找到作用域`foo(..)`中同时存在`a` 和 `b`，然后就不会去寻找外层的 `b`了。因此，我们输出"1 3" 而不是通常情况下的"1 2"。

**Note:** In this example, for simplicity's sake, the string of "code" we pass in was a fixed literal. But it could easily have been programmatically created by adding characters together based on your program's logic. `eval(..)` is usually used to execute dynamically created code, as dynamically evaluating essentially static code from a string literal would provide no real benefit to just authoring the code directly.

**注意：** 在这个例子中，为了简单的缘故，我们传入的"代码"字符串是固定的字面量。但是也可以很简单的按照计划根据你的代码逻辑创建将字符组成到一起的代码。`eval(..)`通常用在执行动态创建的代码，因为动态解析完全从字符串字面量来的静态代码并不会比直接写代码带来真正的好处。

By default, if a string of code that `eval(..)` executes contains one or more declarations (either variables or functions), this action modifies the existing lexical scope in which the `eval(..)` resides. Technically, `eval(..)` can be invoked "indirectly", through various tricks (beyond our discussion here), which causes it to instead execute in the context of the global scope, thus modifying it. But in either case, `eval(..)` can at runtime modify an author-time lexical scope.

默认情况下,如果`eval(..)`执行的代码字符串包含一个或者多个声明(不论变量或者方法),这个操作会修改`eval(..)`所在的已存在的词法作用域。技术上来讲,`eval(..)`可以间接的被调用，通过各种各样的技巧(超出我们这里的讨论),使他在全局上下文里执行，然后修改他。但是在任何一种情况，`eval(..)`可以在运行时修改书写时决定的词法作用域。

**Note:** `eval(..)` when used in a strict-mode program operates in its own lexical scope, which means declarations made inside of the `eval()` do not actually modify the enclosing scope.

**注意:** 当`eval(..)`在严格模式中被使用操作它自己的词法作用域，这意味着`eval()`内部声明的变量不能真正修改包围的作用域。

```js
function foo(str) {
   "use strict";
   eval( str );
   console.log( a ); // ReferenceError: a is not defined
}

foo( "var a = 2" );
```

There are other facilities in JavaScript which amount to a very similar effect to `eval(..)`. `setTimeout(..)` and `setInterval(..)` *can* take a string for their respective first argument, the contents of which are `eval`uated as the code of a dynamically-generated function. This is old, legacy behavior and long-since deprecated. Don't do it!

在JavaScript中还有其他的技巧可以做到和`eval(..)`非常相似的影响。`setTimeout(..)` 和 `setInterval(..)` *可以* 将字符串分别作为第一个参数，他的内容会被`eval`作为是动态生成的方法代码。这个是一个老的,传统的行为,早就不被赞成。所以别这么做。

The `new Function(..)` function constructor similarly takes a string of code in its **last** argument to turn into a dynamically-generated function (the first argument(s), if any, are the named parameters for the new function). This function-constructor syntax is slightly safer than `eval(..)`, but it should still be avoided in your code.

`new Function(..)`构造方法相似的在他的**最后**一个参数需要一个代码字符串然后转换成动态生成方法(如果存在前面的参数,会将他们作为参数传入到新的方法中去)。这个构造方法语法比`eval(..)`稍微安全一点,但是任然避免在你的代码中使用。

The use-cases for dynamically generating code inside your program are incredibly rare, as the performance degradations are almost never worth the capability.

在你的代码中动态生成代码的用例是极其罕见的,对性能的影响也让他的能力几乎没有任何价值。

### `with`

The other frowned-upon (and now deprecated!) feature in JavaScript which cheats lexical scope is the `with` keyword. There are multiple valid ways that `with` can be explained, but I will choose here to explain it from the perspective of how it interacts with and affects lexical scope.

另外一个让人疑惑(现在已经极不赞成)的在JavaScript中用于欺骗词法作用域的特性是 `with` 关键字。让`with`可以被解释有好几种有效的方式，但是我会选择用他是和词法作用域如何交流和如何影响词法作用域的角度来解释他。

`with` is typically explained as a short-hand for making multiple property references against an object *without* repeating the object reference itself each time.

`with`的通常的解释是对多属性引用的简写相反的引用对象就不用每次都重复对象。

For example:

举例来说：

```js
var obj = {
	a: 1,
	b: 2,
	c: 3
};

// more "tedious" to repeat "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// "easier" short-hand
with (obj) {
	a = 3;
	b = 4;
	c = 5;
}
```

However, there's much more going on here than just a convenient short-hand for object property access. Consider:

然而，这里有更多的事发生了而不是仅仅方便了对对象的存取简写。考虑一下：

```js
function foo(obj) {
	with (obj) {
		a = 2;
	}
}

var o1 = {
	a: 3
};

var o2 = {
	b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- Oops, leaked global!
```

In this code example, two objects `o1` and `o2` are created. One has an `a` property, and the other does not. The `foo(..)` function takes an object reference `obj` as an argument, and calls `with (obj) { .. }` on the reference. Inside the `with` block, we make what appears to be a normal lexical reference to a variable `a`, an LHS reference in fact (see Chapter 1), to assign to it the value of `2`.

在这个代码例子中，两个对象`o1` 和 `o2`被创建。一个对象有 `a` 属性，然后另外一个没有。方法 `foo(..)` 将对象`obj`引用作为他的参数，然后执行`with (obj) { .. }`对obj引用。在 `with` 段落中，我们使变量 `a`在一个似乎是平常的词法作用域中，但事实上是一个LHS引用(见第一章)，然后对它赋值为 `2`。

When we pass in `o1`, the `a = 2` assignment finds the property `o1.a` and assigns it the value `2`, as reflected in the subsequent `console.log(o1.a)` statement. However, when we pass in `o2`, since it does not have an `a` property, no such property is created, and `o2.a` remains `undefined`.

当我们传入`o1`，`a = 2`赋值找到属性`o1.a`然后将他赋值`2`，然后在随后的 `console.log(o1.a)`表达式被显示。然而,当我们传入`o2`,之前它没有`a`属性,没有属性被创建,`o2.a`仍然是`undefined`。

But then we note a peculiar side-effect, the fact that a global variable `a` was created by the `a = 2` assignment. How can this be?

但是随后我们注意到一个怪异的副作用，全局变量 `a` 被`a = 2`赋值操作创建了。这是怎么回事？

The `with` statement takes an object, one which has zero or more properties, and **treats that object as if *it* is a wholly separate lexical scope**, and thus the object's properties are treated as lexically defined identifiers in that "scope".

`with`表达式得到一个对象，对象可能有0个或者多个属性，然后**对待那个对象为一个完全隔离的词法作用域**，因此对象的属性被当成标识符在那个"作用域"中被声明。

**Note:** Even though a `with` block treats an object like a lexical scope, a normal `var` declaration inside that `with` block will not be scoped to that `with` block, but instead the containing function scope.

**注意:** 即使一个`with`区块像词法作用域一样对待一个对象，在那个`with`区块的一个普通的`var`声明不会被认为是`with`区块的作用域,而是在包含他的方法作用域中。

While the `eval(..)` function can modify existing lexical scope if it takes a string of code with one or more declarations in it, the `with` statement actually creates a **whole new lexical scope** out of thin air, from the object you pass to it.

而如果一个代码字符串有一个或者多个声明`eval(..)`方法可以修改已经存在的词法作用域，事实上`with`表达式通过你传入的对象凭空创造了一个 **完全全新的词法作用域**。

Understood in this way, the "scope" declared by the `with` statement when we passed in `o1` was `o1`, and that "scope" had an "identifier" in it which corresponds to the `o1.a` property. But when we used `o2` as the "scope", it had no such `a` "identifier" in it, and so the normal rules of LHS identifier look-up (see Chapter 1) occurred.

了解了这种方式,"作用域"在当我们传入`o1`对象是被`with`表达式声明，那个"作用域"有一个"识别符"和`o1.a`属性相符合。但是当我们使用`o2`作为"作用域"，里面并没有`a`"标识符"，所以一个普通的LHS标识符引用(见第一章)被触发了。

Neither the "scope" of `o2`, nor the scope of `foo(..)`, nor the global scope even, has an `a` identifier to be found, so when `a = 2` is executed, it results in the automatic-global being created (since we're in non-strict mode).

不论是`o2`"作用域"，还是`foo(..)`作用域,甚至全局作用于中都没有找到`a`识别符，所以当`a = 2`被执行了，它导致一个自动的全局作用域被创建(前提我们在非严格模式下)。

It is a strange sort of mind-bending thought to see `with` turning, at runtime, an object and its properties into a "scope" *with* "identifiers". But that is the clearest explanation I can give for the results we see.

在运行时，一个对象和他的属性在"作用域"中 *作为* "标识符",这种`with`的运行机制可能有些奇怪。但是这已经是我能给出的对于我们看到的结果最清楚的解释。

**Note:** In addition to being a bad idea to use, both `eval(..)` and `with` are affected (restricted) by Strict Mode. `with` is outright disallowed, whereas various forms of indirect or unsafe `eval(..)` are disallowed while retaining the core functionality.

**注意:** 除了不建议使用以外,`eval(..)` 和 `with`在严格模式下都是受影响(受限)的。`with`是完全不被允许的，虽然保留了核心的功能但是各种形式直接或者不安全的`eval(..)`都是不被允许的。

### Performance


### 效率

Both `eval(..)` and `with` cheat the otherwise author-time defined lexical scope by modifying or creating new lexical scope at runtime.

`eval(..)` 和 `with`都是在运行时通过修改或者创建新的词法作用域的方式欺骗其他书写时定义的词法作用域。

So, what's the big deal, you ask? If they offer more sophisticated functionality and coding flexibility, aren't these *good* features? **No.**

所以,你问这有什么大不了？如果他们能提供更复杂的功能性和代码的灵活性，他们不是 *很好* 的特性吗？ **不**

The JavaScript *Engine* has a number of performance optimizations that it performs during the compilation phase. Some of these boil down to being able to essentially statically analyze the code as it lexes, and pre-determine where all the variable and function declarations are, so that it takes less effort to resolve identifiers during execution.

JavaScript *引擎* 在执行编译阶段期间会诱几个效率的优化。其中的一些可以归结为让他们有能力从本质上根据词法对代码进行静态分析，提前查明变量和方法在哪里声明，所以在执行期间解析标识符就会更小的影响性能。

But if the *Engine* finds an `eval(..)` or `with` in the code, it essentially has to *assume* that all its awareness of identifier location may be invalid, because it cannot know at lexing time exactly what code you may pass to `eval(..)` to modify the lexical scope, or the contents of the object you may pass to `with` to create a new lexical scope to be consulted.

但是如果引擎从代码中发现`eval(..)` 和 `with`，他本质上会*认为*所有位置上的标识符可能是无效的，因为他无法准确的知道在词法分析时间你会将什么代码传入到`eval(..)`中从而改变词法作用域，或者你传入到`with`中的对象的内容会创建一个新的词法作用域用来被查询。

In other words, in the pessimistic sense, most of those optimizations it *would* make are pointless if `eval(..)` or `with` are present, so it simply doesn't perform the optimizations *at all*.

换句话来说,在悲观的情况下，如果`eval(..)`或者`with`出现大部分的优化都 *将* 会变的没有意义，所以索性就 *一点* 都不优化了。

Your code will almost certainly tend to run slower simply by the fact that you include an `eval(..)` or `with` anywhere in the code. No matter how smart the *Engine* may be about trying to limit the side-effects of these pessimistic assumptions, **there's no getting around the fact that without the optimizations, code runs slower.**

事实上不论你将`eval(..)`或者`with`包含在代码的任何地方，你的代码几乎肯定会变的运行缓慢。不论引擎多聪明也许它会试图限制悲观假设的影响，**在不优化性能的情况下,代码运行会变慢的这个事实是无法避免的。**

## Review (TL;DR)

## 回顾 (TL;DR)

Lexical scope means that scope is defined by author-time decisions of where functions are declared. The lexing phase of compilation is essentially able to know where and how all identifiers are declared, and thus predict how they will be looked-up during execution.

词法作用域的意思是作用域的定义在书写时方法在哪里声明决定。在编译的语法分析阶段实质上需要有能力知道所有的标识符在哪里和如何声明,从而决定在执行阶段会如何查看他们。

Two mechanisms in JavaScript can "cheat" lexical scope: `eval(..)` and `with`. The former can modify existing lexical scope (at runtime) by evaluating a string of "code" which has one or more declarations in it. The latter essentially creates a whole new lexical scope (again, at runtime) by treating an object reference *as* a "scope" and that object's properties as scoped identifiers.

有两种机制可以"欺骗"词法作用域:`eval(..)` 和 `with`。前者可以通过对有一个或者多个声明在里面的"代码"字符串进行求值从而修改已经存在的词法作用域(在运行时)。后者实质上是通过对一个对象引用*当做*一个"作用域"然后将对象的属性作为*作用域*的标识符来创建一个全新的词法作用域(在运行时)。

The downside to these mechanisms is that it defeats the *Engine*'s ability to perform compile-time optimizations regarding scope look-up, because the *Engine* has to assume pessimistically that such optimizations will be invalid. Code *will* run slower as a result of using either feature. **Don't use them.**

这些机制的缺点是他阻挠了引擎在执行编译时对作用域查找做的优化能力，因为引擎会悲观的假设所有的优化都会无效。不论你使用哪个特性结果都是代码*将会*运行的缓慢，**不要使用它们。**
