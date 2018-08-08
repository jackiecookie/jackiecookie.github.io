---
layout: post
title: 'You Don''t Know JS: this & Object Prototypes-Chapter 1: this Or That?'
date: 2017-07-29 12:18:17
tags:
---
# You Don't Know JS: *this* & Object Prototypes
# Chapter 1: `this` Or That?

# 你不知道的JS： *this* & Object Prototypes
# 第一章: `this` 或者 That?

One of the most confused mechanisms in JavaScript is the `this` keyword. It's a special identifier keyword that's automatically defined in the scope of every function, but what exactly it refers to bedevils even seasoned JavaScript developers.

JavaScript中最令人困惑的其中之一机制就是`this`关键字。这是一个特别的标识符关键字他会自动在每一个方法作用域中定义，但是他到底引用着什么长期困扰着JavaScript开发者甚至是老练的开发者。

> Any sufficiently *advanced* technology is indistinguishable from magic. -- Arthur C. Clarke

> 任何足够的*先进*的技术和魔法没什么区别。 -- Arthur C. Clarke

JavaScript's `this` mechanism isn't actually *that* advanced, but developers often paraphrase that quote in their own mind by inserting "complex" or "confusing", and there's no question that without lack of clear understanding, `this` can seem downright magical in *your* confusion.

JavaScript的`this`机制并没有真正的*那么*高级，但是开发者经常将那个引用解释就好像在他们的脑子里插入了"复杂"或者"疑惑"，缺乏了清楚的理解，`this`在*你的*疑惑里看起来就像是个十足的魔法这点事毫无疑问的。

**Note:** The word "this" is a terribly common pronoun in general discourse. So, it can be very difficult, especially verbally, to determine whether we are using "this" as a pronoun or using it to refer to the actual keyword identifier. For clarity, I will always use `this` to refer to the special keyword, and "this" or *this* or this otherwise.

**注意:** 单词"this"在现代话语里是一个非常常见的发音。所以，不论是用"this"作为发音或者用它来引用一个真正的关键字标识符，这个可以非常难，特别是口头。为了更清楚，我将会使用`this`来引用一个特殊的关键字，除此之外用"this" 或者 *this* 或者 this。

## Why `this`?

## 为什么`this`?

If the `this` mechanism is so confusing, even to seasoned JavaScript developers, one may wonder why it's even useful? Is it more trouble than it's worth? Before we jump into the *how*, we should examine the *why*.

如果`this`机制是那么的令人疑惑，即使是老练的JavaScript开发者，你可能会好奇为什么他还那么有用?他会带来更多的麻烦还是益处？在我们跳到*如何*之前，我们需要搞清楚*为什么*。

Let's try to illustrate the motivation and utility of `this`:

让我们来试图举例说明`this`的行为动机(motivation:to be the reason why sth in a particular way)和用处：

```js
function identify() {
	return this.name.toUpperCase();
}

function speak() {
	var greeting = "Hello, I'm " + identify.call( this );
	console.log( greeting );
}

var me = {
	name: "Kyle"
};

var you = {
	name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER
```

If the *how* of this snippet confuses you, don't worry! We'll get to that shortly. Just set those questions aside briefly so we can look into the *why* more clearly.

如果这个段代码*如果工作*的使你迷惑，别担心！你马上就会知道。先将这些问题短暂的放在一边这样我们可以把*为什么*看的更清楚。

This code snippet allows the `identify()` and `speak()` functions to be re-used against multiple *context* (`me` and `you`) objects, rather than needing a separate version of the function for each object.

这段代码允许`identify()` 和 `speak()`方法重复使用多个*上下文环境*(`me` 和 `you`)对象，而不是需要对每个对象有一个独立版本的方法。

Instead of relying on `this`, you could have explicitly passed in a context object to both `identify()` and `speak()`.

你可以明确的传入一个上下文对象给`identify()` 和 `speak()`，来代替依赖`this`。

```js
function identify(context) {
	return context.name.toUpperCase();
}

function speak(context) {
	var greeting = "Hello, I'm " + identify( context );
	console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```

However, the `this` mechanism provides a more elegant way of implicitly "passing along" an object reference, leading to cleaner API design and easier re-use.

然而，`this`机制提供了一个更优雅的方式来隐晦的*传递*一个对象的引用，使API更清爽更易复用。

The more complex your usage pattern is, the more clearly you'll see that passing context around as an explicit parameter is often messier than passing around a `this` context. When we explore objects and prototypes, you will see the helpfulness of a collection of functions being able to automatically reference the proper context object.

你使用的模式越复杂，你就会越清楚的看到，传递上下文作为明确的参数通常比传递一个`this`上下文来的更混乱。当我们开始探索对象和原型，你将会看到一个方法集合能够自动引用合适的上下文对象是多么有用。

## Confusions

## 混淆

We'll soon begin to explain how `this` *actually* works, but first we must  dispel some misconceptions about how it *doesn't* actually work.

我们将很快开始探索`this`*实际上* 是如何工作的，但是首先我们必须消除一些错误的观念关于他实际上*不是*如何工作的。

The name "this" creates confusion when developers try to think about it too literally. There are two meanings often assumed, but both are incorrect.

当开发人员试图把"this"想的太字面化这个名字会产生混淆。有两种意思经常被假设，但是他们都是不正确的。

### Itself

### 他自己

The first common temptation is to assume `this` refers to the function itself. That's a reasonable grammatical inference, at least.

第一种常见的诱导是假设`this`引用的方法本身。至少这是一个有原因的符合语法的推论。

Why would you want to refer to a function from inside itself? The most common reasons would be things like recursion (calling a function from inside itself) or having an event handler that can unbind itself when it's first called.

为什么你希望一个方法从内部引用他自己？最常见的原因将会类似重复调用(从方法内部调用自己)或者一个事件管理器可以自己在第一次执行的解绑自己。

Developers new to JS's mechanisms often think that referencing the function as an object (all functions in JavaScript are objects!) lets you store *state* (values in properties) between function calls. While this is certainly possible and has some limited uses, the rest of the book will expound on many other patterns for *better* places to store state besides the function object.

开发者一开始接触JS的机制经常会引用方法作为一个对象(所有的方法在JavaScript中都是对象！)让你在方法调用时存储*状态*(将方法存储在属性上)。然而这确实是有可能的还有一些受局限的用处，书的剩下部分将会详细解释许多其他的模式在方法对象之间提供*更好*的地方来存储状态。

But for just a moment, we'll explore that pattern, to illustrate how `this` doesn't let a function get a reference to itself like we might have assumed.

但是给一点时间，我们将会探索那个模式，来举例说明`this`如何不会像我们假设的那样让一个方法获得他本身的引用。

Consider the following code, where we attempt to track how many times a function (`foo`) was called:

考虑一下接下来打代码，我们试图来跟踪一个方法(`foo`)被调用了几次：

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 0 -- WTF?
```

`foo.count` is *still* `0`, even though the four `console.log` statements clearly indicate `foo(..)` was in fact called four times. The frustration stems from a *too literal* interpretation of what `this` (in `this.count++`) means.

`foo.count`*仍然* 是0，即使通过了四次`console.log`表达式清楚的指出了`foo(..)`事实上被调用了四次。这个令人沮丧的根源是对(`this.count++`的)`this`的解释*太过于字面化*。

When the code executes `foo.count = 0`, indeed it's adding a property `count` to the function object `foo`. But for the `this.count` reference inside of the function, `this` is not in fact pointing *at all* to that function object, and so even though the property names are the same, the root objects are different, and confusion ensues.

当代码执行`foo.count = 0`，事实上是将一个属性`count`附加到方法对象`foo`上。但是`this.count`引用在方法的内部，`this`事实上*根本*不是指向的方法对象，所以即使属性名称相同，根本的对象还是不一样，因此产生了混淆。

**Note:** A responsible developer *should* ask at this point, "If I was incrementing a `count` property but it wasn't the one I expected, which `count` *was* I incrementing?" In fact, were she to dig deeper, she would find that she had accidentally created a global variable `count` (see Chapter 2 for *how* that happened!), and it currently has the value `NaN`. Of course, once she identifies this peculiar outcome, she then has a whole other set of questions: "How was it global, and why did it end up `NaN` instead of some proper count value?" (see Chapter 2).

**注意:** 一个负责的开发者*应该*问这么个问题，"如果我增加的`count`属性并不是一个我期望增加的一个属性，哪个`count`才是我增加的？"事实上，如果他挖的更深入一点，他会发现他意外的创建了一个全局变量`count`(他是如何发生的见第二章)，现在的值是`NaN`。当然，一旦他意识到了这个奇怪的结果，他接下来会有一堆其他的问题:"为什么是全局,为什么他最后是`NaN`而不是其他更合适的可数值？"(见第二章)。

Instead of stopping at this point and digging into why the `this` reference doesn't seem to be behaving as *expected*, and answering those tough but important questions, many developers simply avoid the issue altogether, and hack toward some other solution, such as creating another object to hold the `count` property:

相反的停止这个相反然后去深究为什么`this`引用似乎跟*期望*的举止不太一样，然后回答这些困难而且重要的问题，许多开发者会简单的回避这些问题，然后开辟出其他的解决方案，类似创建另外对象来保存`count`属性:

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	data.count++;
}

var data = {
	count: 0
};

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( data.count ); // 4
```

While it is true that this approach "solves" the problem, unfortunately it simply ignores the real problem -- lack of understanding what `this` means and how it works -- and instead falls back to the comfort zone of a more familiar mechanism: lexical scope.

虽然这的确"解决"问题的一种类似方式，不幸的是这让你忽略了真正的问题 -- 对`this`是什么意思以及他是如何工作的的缺乏理解 -- 相反的退回到舒适区使用更熟悉的机制:词法作用域。

**Note:** Lexical scope is a perfectly fine and useful mechanism; I am not belittling the use of it, by any means (see *"Scope & Closures"* title of this book series). But constantly *guessing* at how to use `this`, and usually being *wrong*, is not a good reason to retreat back to lexical scope and never learn *why* `this` eludes you.

**注意:** 词法作用域是一个非常好并且有用的机制;我无论如何不是贬低他的用处(见 这个系列的*"作用域&闭包"* 标头 )。但是不断的*猜测*如何使用`this`，这经常会*犯错*，这不是一个好的理由来撤退回词法作用域然后永远不学*为什么*`this`总是不按照你说的做。(eludes you:you are not able to achieve it.)

To reference a function object from inside itself, `this` by itself will typically be insufficient. You generally need a reference to the function object via a lexical identifier (variable) that points at it.

从方法内部引用一个方法对象，`this`理解为他本身通常是不充分的。你通常需要经由一个词法标识符(变量)来引用一个方法。

Consider these two functions:

考虑这两个方法:

```js
function foo() {
	foo.count = 4; // `foo` refers to itself
}

setTimeout( function(){
	// anonymous function (no name), cannot
	// refer to itself
}, 10 );
```

In the first function, called a "named function", `foo` is a reference that can be used to refer to the function from inside itself.

第一个方法，被叫做"有名字的方法"，`foo`可以使用来从方法内部引用方法的一个引用。

But in the second example, the function callback passed to `setTimeout(..)` has no name identifier (so called an "anonymous function"), so there's no proper way to refer to the function object itself.

但是第二个例子，传入到`setTimeout(..)`的回调方法是没有名字标识符的(所以被叫做"匿名方法")，所以没有合适的方法来引用方法对象本身。

**Note:** The old-school but now deprecated and frowned-upon `arguments.callee` reference inside a function *also* points to the function object of the currently executing function. This reference is typically the only way to access an anonymous function's object from inside itself. The best approach, however, is to avoid the use of anonymous functions altogether, at least for those which require a self-reference, and instead use a named function (expression). `arguments.callee` is deprecated and should not be used.

**注意:** `arguments.callee`是老式而且现在已经不被推荐而且让人皱眉的引用它在一个方法内部*仍然*指向当前执行方法的方法对象。这个引用通常是唯一一种可以从匿名方法内部读取方法对象的方式。最接近的方式，然而，总而言之还是要避免使用匿名方法，至少对于那些需要自己引用自己的方法来说，可以相反的使用有名字的方法(表达式)。`arguments.callee`是不被推荐的应该不要使用。

So another solution to our running example would have been to use the `foo` identifier as a function object reference in each place, and not use `this` at all, which *works*:

所以让我们的例子能够工作的其他的解决方案是使用`foo`标识符作为一个方法的引用在每个地方使用，完全不使用`this`，也可以*工作*。

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	foo.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

However, that approach similarly side-steps *actual* understanding of `this` and relies entirely on the lexical scoping of variable `foo`.

然而，这种处理方式就是类似回避了*真正*理解`this`然后完全依赖词法作用域的变量`foo`。

Yet another way of approaching the issue is to force `this` to actually point at the `foo` function object:

处理这个问题的另外一个方式是强迫`this`真正的指向`foo`方法对象：

```js
function foo(num) {
	console.log( "foo: " + num );

	// keep track of how many times `foo` is called
	// Note: `this` IS actually `foo` now, based on
	// how `foo` is called (see below)
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		// using `call(..)`, we ensure the `this`
		// points at the function object (`foo`) itself
		foo.call( foo, i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

**Instead of avoiding `this`, we embrace it.** We'll explain in a little bit *how* such techniques work much more completely, so don't worry if you're still a bit confused!

**相反的回避`this`，我们选择拥抱他。** 我们将会对类似这样的技巧是如何工作的进行一点解释让他有更完整的解释，所以如果你仍然还有一点疑惑不要担心！

### Its Scope

### 他的作用域

The next most common misconception about the meaning of `this` is that it somehow refers to the function's scope. It's a tricky question, because in one sense there is some truth, but in the other sense, it's quite misguided.

下一个关于`this`的意思最常见的错误观念是 `this`以某种方式引用方法的作用域。这是一个棘手的问题，因为有一个意思他是对的，但是另外一个意思，他完全是被误导的。

To be clear, `this` does not, in any way, refer to a function's **lexical scope**. It is true that internally, scope is kind of like an object with properties for each of the available identifiers. But the scope "object" is not accessible to JavaScript code. It's an inner part of the *Engine*'s implementation.

说的更清楚一些,`this`不管以何种方式都不会是对一个方法的**词法作用域**的引用。在内部他是对的，作用域是一种类似一个将各种标识符作为属性的对象。但是作用域"对象"对JavaScript代码来说是不可读取的、他是一个*引擎*的内部工具。

Consider code which attempts (and fails!) to cross over the boundary and use `this` to implicitly refer to a function's lexical scope:

考虑这个代码它试图(然后失败)穿过边界然后使用`this`隐晦的引用一个方法的词法作用域：

```js
function foo() {
	var a = 2;
	this.bar();
}

function bar() {
	console.log( this.a );
}

foo(); //undefined
```

There's more than one mistake in this snippet. While it may seem contrived, the code you see is a distillation of actual real-world code that has been exchanged in public community help forums. It's a wonderful (if not sad) illustration of just how misguided `this` assumptions can be.

这个代码片段有不止一处的错误。虽然这可能看起来是人为的，但你看到的代码是一个真正的从公开社区互助讨论区里截取来的代码。这个是可以很好的用来举例说明对`this`的假设是如何误导的。

Firstly, an attempt is made to reference the `bar()` function via `this.bar()`. It is almost certainly an *accident* that it works, but we'll explain the *how* of that shortly. The most natural way to have invoked `bar()` would have been to omit the leading `this.` and just make a lexical reference to the identifier.

首先一个尝试通过`this.bar()`来引用`bar()`方法。这个几乎可以确定是 *意外* 它居然成功了，但是我们将简短的解释他是*如何*成功的。调用`bar()`最自然的方式是通过忽略开始的`this.`然后只需要创建一个对标识符的词法引用。

However, the developer who writes such code is attempting to use `this` to create a bridge between the lexical scopes of `foo()` and `bar()`, so that `bar()` has access to the variable `a` in the inner scope of `foo()`. **No such bridge is possible.** You cannot use a `this` reference to look something up in a lexical scope. It is not possible.

然而，写类似代码的开发者尝试使用`this`来创建一个位于`foo()`和`bar()`词法作用域的桥，所以`bar()`可以在`foo()`的内部作用域读取变量`a`。**这样的桥是不存在的** 你不可以在一个在词法作用域中用`this`引用来查找东西。这是不可能的。

Every time you feel yourself trying to mix lexical scope look-ups with `this`, remind yourself: *there is no bridge*.

每次你感觉你自己尝试使用`this`在混合的词法作用域中查找，提醒你自己: *他们之间没有桥*。

## What's `this`?

## 什么是`this`?

Having set aside various incorrect assumptions, let us now turn our attention to how the `this` mechanism really works.

将各种不正确的假设放在一边，让我们现在把我们的注意力转到`this`机制真正是如何工作的上面来。

We said earlier that `this` is not an author-time binding but a runtime binding. It is contextual based on the conditions of the function's invocation. `this` binding has nothing to do with where a function is declared, but has instead everything to do with the manner in which the function is called.

我们之前说过`this`不是一个书写时绑定而是一个运行的绑定。他是以方法调用环境上下文为基础的。`this`绑定跟放在在哪里调用无关，但是相反的和方法被什么调用有关。

When a function is invoked, an activation record, otherwise known as an execution context, is created. This record contains information about where the function was called from (the call-stack), *how* the function was invoked, what parameters were passed, etc. One of the properties of this record is the `this` reference which will be used for the duration of that function's execution.

当一个方法被调用，会创建一个激活记录，另外也被叫做一个执行上下文。这个记录包含的信息有方法从哪里被调用(调用堆栈)，方法*如何*被调用，什么参数被传入，等等。这个记录的其中一个属性是`this`引用他将会在方法的执行期间被使用。

In the next chapter, we will learn to find a function's **call-site** to determine how its execution will bind `this`.

在下一章，我们会学习找到一个方法的**调用位置**来确定执行将会如何绑定`this`。

## Review (TL;DR)

## 复习 (TL;DR)

`this` binding is a constant source of confusion for the JavaScript developer who does not take the time to learn how the mechanism actually works. Guesses, trial-and-error, and blind copy-n-paste from Stack Overflow answers is not an effective or proper way to leverage *this* important `this` mechanism.

`this`绑定对那些没有花时间去学习这个机制是如何真正工作的人来说是会一直困惑的。猜测，实验然后出错，闭着眼睛从Stack Overflow拷贝粘贴不是一个有效或者更好的方式来利用这个重要的 `this` 机制。

To learn `this`, you first have to learn what `this` is *not*, despite any assumptions or misconceptions that may lead you down those paths. `this` is neither a reference to the function itself, nor is it a reference to the function's *lexical* scope.

要学`this`，你首先要学`this`不是什么，不管哪种假设或者错误的观点可能会使你走到错误的路上。`this`既不是对方法自己的一个引用，也不是对方法的*词法*作用域的一个引用。

`this` is actually a binding that is made when a function is invoked, and *what* it references is determined entirely by the call-site where the function is called.

`this`实际上是当方法被执行时进行的绑定，而他引用着什么整个取决于调用的位置也就是方法在哪里被执行。
