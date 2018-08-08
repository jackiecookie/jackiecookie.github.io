# You Don't Know JS: Scope & Closures
# Chapter 5: Scope Closure

# 你不知道的JS：作用域 & 闭包
# 第五章：作用域闭包

We arrive at this point with hopefully a very healthy, solid understanding of how scope works.

我们希望我们能带着健全，扎实的对作用域如何工作的理解来带这里。

We turn our attention to an incredibly important, but persistently elusive, *almost mythological*, part of the language: **closure**. If you have followed our discussion of lexical scope thus far, the payoff is that closure is going to be, largely, anticlimactic, almost self-obvious. *There's a man behind the wizard's curtain, and we're about to see him*. No, his name is not Crockford!

我们把注意力转到一个极其重要，但是反复出现且难以察觉，几乎是语言神话般的一部分:**闭包**。如果你关注了我们到目前为止对词法作用域的讨论，结果就是闭包将会变的有些扫兴，几乎是显而易见的。*一个男人躲在巫师的窗帘后面，我们要去见他*。不他的名字不是Crockford。

If however you have nagging questions about lexical scope, now would be a good time to go back and review Chapter 2 before proceeding.

然而如果你有一些关于词法作用域难以搞懂的问题，现在在处理之前会是一个好时机去回顾第二章的内容。

## Enlightenment

## 启发

For those who are somewhat experienced in JavaScript, but have perhaps never fully grasped the concept of closures, *understanding closure* can seem like a special nirvana that one must strive and sacrifice to attain.

对那些稍微对JavaScript有点经验的人，但是也许从来没有完全领悟闭包概念的人，*理解闭包* 似乎有点像一个我们必须努力和牺牲去达到的特别的境界。

I recall years back when I had a firm grasp on JavaScript, but had no idea what closure was. The hint that there was *this other side* to the language, one which promised even more capability than I already possessed, teased and taunted me. I remember reading through the source code of early frameworks trying to understand how it actually worked. I remember the first time something of the "module pattern" began to emerge in my mind. I remember the *a-ha!* moments quite vividly.

我回忆几年之前当我对JavaScript掌握的挺扎实，但我还是不知道闭包是什么。这暗示着语言的*另外一边*，他承诺了比我准备处理的更多的能力，取笑和戏弄我。我记得我通过读早起框架的源代码来试图理解它实际上是如何工作的。我记得第一次"模块模式的"一些东西开始出现在我脑海里。我记得*阿哈*的瞬间非常的生动。

What I didn't know back then, what took me years to understand, and what I hope to impart to you presently, is this secret: **closure is all around you in JavaScript, you just have to recognize and embrace it.** Closures are not a special opt-in tool that you must learn new syntax and patterns for. No, closures are not even a weapon that you must learn to wield and master as Luke trained in The Force.

什么是我那个时候不知道的，什么让我花了几年的时间来理解，什么是我希望最先传授给你的，是这个秘密：**在JavaScript中到处都有闭包，你只需要认出他然后拥抱他。** 闭包不是一个需要你为他学习新的语法和模式的特别的选择性使用的工具。不，闭包甚至不是一个你必须学着使用的武器也不需要像Luke一样训练像大师一样使用这种武力。

Closures happen as a result of writing code that relies on lexical scope. They just happen. You do not even really have to intentionally create closures to take advantage of them. Closures are created and used for you all over your code. What you are *missing* is the proper mental context to recognize, embrace, and leverage closures for your own will.

闭包作为书写代码的结果依赖于词法作用域。他就这样发生了。你甚至不是有意的创造了闭包然后在它身上获得益处。闭包在你的代码的各个地方被创建和使用。你缺失是更好的思维环境去识别，拥抱，和影响闭包来做你自己想做的事。

The enlightenment moment should be: **oh, closures are already occurring all over my code, I can finally *see* them now.** Understanding closures is like when Neo sees the Matrix for the first time.

这个开光的瞬间应该是:**噢，闭包已经在我代码的到处执行了，现在我终于可以*看见*他*了** 理解闭包就像第一次Neo看见Matrix的时候。

## Nitty Gritty

## 重要细节

OK, enough hyperbole and shameless movie references.

OK,已经受够了这种夸张且没有羞耻心的电影引用了。

Here's a down-n-dirty definition of what you need to know to understand and recognize closures:

你需要如何理解和识别闭包,这里是定义

> Closure is when a function is able to remember and access its lexical scope even when that function is executing outside its lexical scope.

> 闭包是当一个方法需要记住和存取这个作用域尽管这个方法在这个作用域的外部执行。

Let's jump into some code to illustrate that definition.

让我们跳到某些代码来举例说明这个定义。

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a ); // 2
	}

	bar();
}

foo();
```

This code should look familiar from our discussions of Nested Scope. Function `bar()` has *access* to the variable `a` in the outer enclosing scope because of lexical scope look-up rules (in this case, it's an RHS reference look-up).

经过我们之前讨论的嵌套作用域这个代码应该会看起来有些熟悉。方法`bar()`有*读取*在包围作用域外部的变量`a`因为根据作用域查询规则他可以这么做(在这个例子中，是一个RHS引用查看)。

Is this "closure"?

这就是"闭包"?

Well, technically... *perhaps*. But by our what-you-need-to-know definition above... *not exactly*. I think the most accurate way to explain `bar()` referencing `a` is via lexical scope look-up rules, and those rules are *only* (an important!) **part** of what closure is.

恩,技术上来说...*也许*。但是根据什么是你需要知道关于定义...*不准确*。我想最准确的方式来解释`bar()`对`a`的引用是它根据了词法作用域的查询规则，这些规则*只是*(一个很重要的)闭包的一部分。

From a purely academic perspective, what is said of the above snippet is that the function `bar()` has a *closure* over the scope of `foo()` (and indeed, even over the rest of the scopes it has access to, such as the global scope in our case). Put slightly differently, it's said that `bar()` closes over the scope of `foo()`. Why? Because `bar()` appears nested inside of `foo()`. Plain and simple.

根据一个纯学术的观点，我们说这段代码方法`bar()`有对于作用域`foo()`一个闭包(的确，即使是在其余的作用域中还是可以读取，你像我们例子中的全局作用域)。换句略有不同的话来说，`bar()`关闭了`foo()`作用域。为什么?因为`bar()`出现被嵌套在`foo()`中。简单清楚。

But, closure defined in this way is not directly *observable*, nor do we see closure *exercised* in that snippet. We clearly see lexical scope, but closure remains sort of a mysterious shifting shadow behind the code.

但是，闭包被按照这种方式定义就不是直接为*可观察*了，闭包也不是我们在这段代码中看到的这样使用。我们清楚的看到词法作用域，但是闭包就像是代码背后神秘的快速移动的影子。

Let us then consider code which brings closure into full light:

让我们接下来考虑这段使闭包被照亮显示出来的代码：

```js
function foo() {
	var a = 2;

	function bar() {
		console.log( a );
	}

	return bar;
}

var baz = foo();

baz(); // 2 -- Whoa, closure was just observed, man.
```

The function `bar()` has lexical scope access to the inner scope of `foo()`. But then, we take `bar()`, the function itself, and pass it *as* a value. In this case, we `return` the function object itself that `bar` references.

方法`bar()`有`foo()`作用域内部的词法作用域读取权。但是现在,我们得到`bar()`方法本身，然后把他作为值一样传输。在这个例子中，我们`return``bar`方法对象本身的引用。

After we execute `foo()`, we assign the value it returned (our inner `bar()` function) to a variable called `baz`, and then we actually invoke `baz()`, which of course is invoking our inner function `bar()`, just by a different identifier reference.

在我们执行了`foo()`之后，我们把返回回来(我们内部的方法)的值赋值给了一个变量叫做`baz`，接着我们触发了`baz()`，事实上就是执行了我们的内部方法 `bar()`,只不过使用了另外一个标识符引用。

`bar()` is executed, for sure. But in this case, it's executed *outside* of its declared lexical scope.

可以肯定的是`bar()`被执行了。但是在这个例子中，他在声明作用域的*外面*被执行了。

After `foo()` executed, normally we would expect that the entirety of the inner scope of `foo()` would go away, because we know that the *Engine* employs a *Garbage Collector* that comes along and frees up memory once it's no longer in use. Since it would appear that the contents of `foo()` are no longer in use, it would seem natural that they should be considered *gone*.

`foo()`被执行后，通常情况下我们会猜想整个`foo()`作用域内部会释放，因为我们知道*引擎*使用一个*垃圾回收*来释放之后都不在使用的内存。因为`foo()`的内容看起来不在被使用，似乎自然的他应该被认为是*消失*了。

But the "magic" of closures does not let this happen. That inner scope is in fact *still* "in use", and thus does not go away. Who's using it? **The function `bar()` itself**.

但是闭包的"魔法"不会让这个发生。内部的作用域实际上*仍然*"在使用"，所以不能被释放。谁在使用？ **方法`bar()`本身**。

By virtue of where it was declared, `bar()` has a lexical scope closure over that inner scope of `foo()`, which keeps that scope alive for `bar()` to reference at any later time.

因为他声明的位置，`bar()`在作用域`foo()`内部有一个词法作用域闭包，他使那个作用始终存活着为了 `bar()`能够在后面的时间能够引用。

**`bar()` still has a reference to that scope, and that reference is called closure.**

**`bar()`仍然对那个作用域有一个引用，这个引用被叫做闭包。**

So, a few microseconds later, when the variable `baz` is invoked (invoking the inner function we initially labeled `bar`), it duly has *access* to author-time lexical scope, so it can access the variable `a` just as we'd expect.

所以，几毫秒之后，当变量`baz`被触发(执行我们初始化标记为`bar`的内部方法)，他当然有访问书写时词法作用域的权限，所以他可以像我们希望的一样访问变量`a`。

The function is being invoked well outside of its author-time lexical scope. **Closure** lets the function continue to access the lexical scope it was defined in at author-time.

这个方法在书写时词法作用域的外面仍然可以被执行。**闭包** 使方法可以继续读取在书写时定义的词法作用域。

Of course, any of the various ways that functions can be *passed around* as values, and indeed invoked in other locations, are all examples of observing/exercising closure.

当然，方法可以用各种各样的方式去*传来传去*作为变量，的确可以在其他位置被触发，这都是典型的观察/使用闭包。

```js
function foo() {
	var a = 2;

	function baz() {
		console.log( a ); // 2
	}

	bar( baz );
}
、
function bar(fn) {
	fn(); // look ma, I saw closure!
}
```

We pass the inner function `baz` over to `bar`, and call that inner function (labeled `fn` now), and when we do, its closure over the inner scope of `foo()` is observed, by accessing `a`.

我们把内部的方法`baz`传给`bar`，然后执行这个内部方法(现在被标记成`fn`)，然后当我们这么做的时候，`foo()`内部作用域的闭包通过读取变量`a`被观察了。

These passings-around of functions can be indirect, too.

这种传递方法也可以是间接的。

```js
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log( a );
	}

	fn = baz; // assign `baz` to global variable
}

function bar() {
	fn(); // look ma, I saw closure!
}

foo();

bar(); // 2
```

Whatever facility we use to *transport* an inner function outside of its lexical scope, it will maintain a scope reference to where it was originally declared, and wherever we execute it, that closure will be exercised.

不论我们用了什么设施来*运输*一个内部的作用域运出到他外部的作用域，这都将会保持作用域他原本声明的地方的引用，不管他在哪里被执行，这个闭包都将会被使用。

## Now I Can See

## 现在我看见了

The previous code snippets are somewhat academic and artificially constructed to illustrate *using closure*. But I promised you something more than just a cool new toy. I promised that closure was something all around you in your existing code. Let us now *see* that truth.

前面的代码片段有些某种程度上有点从学术和不自然的结构上来举例说明*使用闭包*。但是我保证你看到不仅仅是一个酷的新玩具。我保证闭包在你存在的代码中到处都有。让我们现在来*看*真相。

```js
function wait(message) {

	setTimeout( function timer(){
		console.log( message );
	}, 1000 );

}

wait( "Hello, closure!" );
```

We take an inner function (named `timer`) and pass it to `setTimeout(..)`. But `timer` has a scope closure over the scope of `wait(..)`, indeed keeping and using a reference to the variable `message`.

我们将一个内部方法(取名为`timer`)传给`setTimeout(..)`。但是`timer`有一个作用域闭包引用着作用域`wait(..)`，事实上保持和引用着变量`message`的引用。

A thousand milliseconds after we have executed `wait(..)`, and its inner scope should otherwise be long gone, that inner function `timer` still has closure over that scope.

一千毫秒之后我们执行了`wait(..)`,要不是内部的方法`timer`仍然有作用域的闭包，否则它内部的作用域应该早就消失了。

Deep down in the guts of the *Engine*, the built-in utility `setTimeout(..)` has reference to some parameter, probably called `fn` or `func` or something like that. *Engine* goes to invoke that function, which is invoking our inner `timer` function, and the lexical scope reference is still intact.

将*引擎*的核心挖的更深，构建工具`setTimeout(..)`有一些参数的引用，可能叫做`fn`或者`func`又或者是类似的名字。*引擎* 会去执行这个方法，也就是我们这里我们内部的`timer`方法，他的词法作用域引用仍然完好无损。

**Closure.**

**闭包。**

Or, if you're of the jQuery persuasion (or any JS framework, for that matter):

或者，如果你是jQuery信仰者(或者任何一个JS框架,关于这一点)：

```js
function setupBot(name,selector) {
	$( selector ).click( function activator(){
		console.log( "Activating: " + name );
	} );
}

setupBot( "Closure Bot 1", "#bot_1" );
setupBot( "Closure Bot 2", "#bot_2" );
```

I am not sure what kind of code you write, but I regularly write code which is responsible for controlling an entire global drone army of closure bots, so this is totally realistic!

我不确定你写的代码时什么类型，但是我经常写的代码来负责控制整个全球的闭包机器人军团，所以这就是现实。

(Some) joking aside, essentially *whenever* and *wherever* you treat functions (which access their own respective lexical scopes) as first-class values and pass them around, you are likely to see those functions exercising closure. Be that timers, event handlers, Ajax requests, cross-window messaging, web workers, or any of the other asynchronous (or synchronous!) tasks, when you pass in a *callback function*, get ready to sling some closure around!

将玩笑放一边，本质上你把方法(读取他们各自词法作用域)作为第一类值然后不管*什么时候*和*在哪里*将他们传来传去，我基本上可以看见这些方法运用了闭包。当你使用timers，事件处理，Ajax请求，跨window的消息，web workers，或者任何一个其他的异步(或者同步)任务，当你传入一个*回调方法*，其实就是将闭包一起甩了过去。

**Note:** Chapter 3 introduced the IIFE pattern. While it is often said that IIFE (alone) is an example of observed closure, I would somewhat disagree, by our definition above.

**注意:** 第三章介绍了IIFE模式。虽然经常说IIFE(独自的)是一个观察闭包的例子，但根据我们的定义，某种程度上我不同意。

```js
var a = 2;

(function IIFE(){
	console.log( a );
})();
```

This code "works", but it's not strictly an observation of closure. Why? Because the function (which we named "IIFE" here) is not executed outside its lexical scope. It's still invoked right there in the same scope as it was declared (the enclosing/global scope that also holds `a`). `a` is found via normal lexical scope look-up, not really via closure.

这个代码"可以工作",但是这不是严格意义上的观察闭包。为什么？因为方法(我们在这里命名为"IIFE")没有在他词法作用域外部执行。他仍然在他声明(包围的/全局作用域仍然保持着`a`)的同一个作用域中被执行。`a`经过普通的词法作用域查看找到，而不是真正的通过闭包找到。

While closure might technically be happening at declaration time, it is *not* strictly observable, and so, as they say, *it's a tree falling in the forest with no one around to hear it.*

虽然闭包技术上发生在声明时，但*不是*严格的可观察，所以，就像他们说的，*一棵树落在了森林里没人看见也没人听见。*

Though an IIFE is not *itself* an example of closure, it absolutely creates scope, and it's one of the most common tools we use to create scope which can be closed over. So IIFEs are indeed heavily related to closure, even if not exercising closure themselves.

尽管一个IIFE*本身*不是闭包的例子，但是他绝对创建作用域，而且他是我们用于创建作用域而且别人无法访问的最常用工具的其中之一。所以IIFE的确很大程度上和闭包相关，即使他自己没有使用闭包。

Put this book down right now, dear reader. I have a task for you. Go open up some of your recent JavaScript code. Look for your functions-as-values and identify where you are already using closure and maybe didn't even know it before.

亲爱的读者，现在合上这本书。我有一个任务给你。去打开你最近写的JavaScript代码。找到你把方法作为参数和找出之前并不知道但使用了闭包的地方。

I'll wait.

我会等你。

Now... you see!

现在... 你看到了！

## Loops + Closure

## 循环 + 闭包

The most common canonical example used to illustrate closure involves the humble for-loop.

一个最常见的典型的例子被使用来说明闭包的是关于最简陋的for循环。

```js
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

**Note:** Linters often complain when you put functions inside of loops, because the mistakes of not understanding closure are **so common among developers**. We explain how to do so properly here, leveraging the full power of closure. But that subtlety is often lost on linters and they will complain regardless, assuming you don't *actually* know what you're doing.

**注意：** 当你把方法放在循环的里面Linters通常会警告，因为不理解闭包造成的错误**在开发人员中很常见**。我们会解释在这里如何做是正确的，充分利用闭包的力量。但是linters经常忽略了这里的微妙之处而且不管怎么样都要警告，就好像你*事实上*不知道你自己在做什么。

The spirit of this code snippet is that we would normally *expect* for the behavior to be that the numbers "1", "2", .. "5" would be printed out, one at a time, one per second, respectively.

这段代码我们通常会*期望*的举止是数字"1", "2", .. "5"会一个接一个一秒接一秒的各自被打印，

In fact, if you run this code, you get "6" printed out 5 times, at the one-second intervals.

事实上，如果你运行这个代码，你会得到"6"每次间隔一秒的被打印了5次。

**Huh?**

**啊?**

Firstly, let's explain where `6` comes from. The terminating condition of the loop is when `i` is *not* `<=5`. The first time that's the case is when `i` is 6. So, the output is reflecting the final value of the `i` after the loop terminates.

首先，让我们解释`6`从哪里来。循环结束的条件是当`i`*不*`<=5`。第一次满足这个条件是当`i`为6的时候。所以，输出是引用`i`在循环结束后最终的值。

This actually seems obvious on second glance. The timeout function callbacks are all running well after the completion of the loop. In fact, as timers go, even if it was `setTimeout(.., 0)` on each iteration, all those function callbacks would still run strictly after the completion of the loop, and thus print `6` each time.

多撇一眼的话这可能会更明显。timeout方法回调会在循环完成之后运行。事实上，作为计时器，即使每次循环为`setTimeout(.., 0)`，所有的方法回调仍然会严格的在循环完成后运行，因此每次输出`6`。

But there's a deeper question at play here. What's *missing* from our code to actually have it behave as we semantically have implied?

但是有一个更深的问题我们要在这里讨论，让我们的代码真正的有像我们语义上暗示的那样的举止我们*缺少*了什么？

What's missing is that we are trying to *imply* that each iteration of the loop "captures" its own copy of `i`, at the time of the iteration. But, the way scope works, all 5 of those functions, though they are defined separately in each loop iteration, all **are closed over the same shared global scope**, which has, in fact, only one `i` in it.

我们缺少的是我们尝试*暗示*每次for循环捕获在每次循环的时候`i`本身的拷贝。但是，作用域是这样工作的，所有的5个方法，尽管在每次循环中定义了独立的方法，所有**都被包在同一个共享全局作用域中**，事实上，只有一个`i`在这个作用域中。

Put that way, *of course* all functions share a reference to the same `i`. Something about the loop structure tends to confuse us into thinking there's something else more sophisticated at work. There is not. There's no difference than if each of the 5 timeout callbacks were just declared one right after the other, with no loop at all.

就因为这种方式。*当然* 所有的方法分享同一个`i`引用。有些关于循环的结构会趋向于误导我们去想有些东西让他更精练的工作。不是的，5个timeout回调一个接一个的声明和有没有循环完全没有区别。

OK, so, back to our burning question. What's missing? We need more ~~cowbell~~ closured scope. Specifically, we need a new closured scope for each iteration of the loop.

OK，所以，回到我们火烧眉毛的问题。到底少了什么？我们需要更多 ~~牛铃声~~ 闭包作用域。确切来说，我们需要给一次循环一个新的闭包作用域。

We learned in Chapter 3 that the IIFE creates scope by declaring a function and immediately executing it.

我们在第三章学到过IIFE通过声明一个方法然后立即执行他来创建作用域。

Let's try:

让我们试一下：

```js
for (var i=1; i<=5; i++) {
	(function(){
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 );
	})();
}
```

Does that work? Try it. Again, I'll wait.

这成功了么？再试一下。我会等着。

I'll end the suspense for you. **Nope.** But why? We now obviously have more lexical scope. Each timeout function callback is indeed closing over its own per-iteration scope created respectively by each IIFE.

我会替你结束这个悬念。**不行。** 但是为什么?我们现在显然有了更多的词法作用域。每个timeout方法回调的确被各自自己的每一次循环执行的IIFE创建的作用域包围起来了。

It's not enough to have a scope to close over **if that scope is empty**. Look closely. Our IIFE is just an empty do-nothing scope. It needs *something* in it to be useful to us.

**如果作用域是空的** 被这样的作用域包围起来是不够的。看仔细了。我们的IIFE只一个空的什么都没做的作用域。需要*某些东西*在里面才会对我们有用。

It needs its own variable, with a copy of the `i` value at each iteration.

需要他们自己的变量，用来每次循环拷贝`i`的值。

```js
for (var i=1; i<=5; i++) {
	(function(){
		var j = i;
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})();
}
```

**Eureka! It works!**

**找到了！成功了！**

A slight variation some prefer is:

有些人更喜欢一个稍微不一样的变形：

```js
for (var i=1; i<=5; i++) {
	(function(j){
		setTimeout( function timer(){
			console.log( j );
		}, j*1000 );
	})( i );
}
```

Of course, since these IIFEs are just functions, we can pass in `i`, and we can call it `j` if we prefer, or we can even call it `i` again. Either way, the code works now.

当然，比起IIFEs只是一个方法，我们现在传入`i`，如果我们喜欢我们可以把他叫做`j`，或者我们可以还是可以取名叫做`i`。不管哪种方法，这个代码可以成功了。

The use of an IIFE inside each iteration created a new scope for each iteration, which gave our timeout function callbacks the opportunity to close over a new scope for each iteration, one which had a variable with the right per-iteration value in it for us to access.

在每一次循环的内部使用一个IIFE替每一次循环创建一个新的作用域，给我们timeout方法回调一个机会在每一次循环闭包一个新的作用域，每一次循环都有一个变量提供一个每一次循环正确的值来读取。

Problem solved!

问题解决了！

### Block Scoping Revisited

### 重温块作用域

Look carefully at our analysis of the previous solution. We used an IIFE to create new scope per-iteration. In other words, we actually *needed* a per-iteration **block scope**. Chapter 3 showed us the `let` declaration, which hijacks a block and declares a variable right there in the block.

仔细看一下我们对前面解决方案的分析。我们使用IIFE为每一次循环创建一个新的作用域。换句话来说，我们实际上*需要*一个每一个循环**块级作用域**。第三章像我们展示了`let`声明，它劫持了一个块然后在块里声明了一个变量。

**It essentially turns a block into a scope that we can close over.** So, the following awesome code "just works":

**这实际上将块转换成我们可以用来闭包的作用域。** 所以，这个了不起的代码"就这样成功了"：

```js
for (var i=1; i<=5; i++) {
	let j = i; // yay, block-scope for closure!
	setTimeout( function timer(){
		console.log( j );
	}, j*1000 );
}
```

*But, that's not all!* (in my best Bob Barker voice). There's a special behavior defined for `let` declarations used in the head of a for-loop. This behavior says that the variable will be declared not just once for the loop, **but each iteration**. And, it will, helpfully, be initialized at each subsequent iteration with the value from the end of the previous iteration.

*但是，这不是全部* (用我最Bob Barker的声音)。`let`声明在for循环的头部使用有一个特别行为定义，这个行为说的是变量将不会仅仅为一次循环声明，**而是每一次循环都声明**。每一次循环按照顺序根据前一次循环完之后的值初始化值，这将会很有帮助。

```js
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
```

How cool is that? Block scoping and closure working hand-in-hand, solving all the world's problems. I don't know about you, but that makes me a happy JavaScripter.

这有多酷？块作用域和闭包手拉着手一起工作，解决世界上所有的问题。我不认识你，但是这使我成为一个开心的JavaScript使用者。

## Modules

## 模块

There are other code patterns which leverage the power of closure but which do not on the surface appear to be about callbacks. Let's examine the most powerful of them: *the module*.

有其他一种代码的模式充分利用闭包的力量但是不像回调一样在表面显示。让我们调查一下最强大的他们：*模块*。

```js
function foo() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}
}
```

As this code stands right now, there's no observable closure going on. We simply have some private data variables `something` and `another`, and a couple of inner functions `doSomething()` and `doAnother()`, which both have lexical scope (and thus closure!) over the inner scope of `foo()`.

就像这个代码标准一样，他们没有可观察的闭包在执行。我们简单有一些私有的数据变量`something` 和 `another`，还有一对内部方法`doSomething()` 和 `doAnother()`，他们都有词法作用域(这样闭包！)在`foo()`内部作用域中。

But now consider:

但是现在考虑一下：

```js
function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

This is the pattern in JavaScript we call *module*. The most common way of implementing the module pattern is often called "Revealing Module", and it's the variation we present here.

这个模式在JavaScript中我们叫做*模块*。

Let's examine some things about this code.

让我们来调查关于这个代码的一些事情。

Firstly, `CoolModule()` is just a function, but it *has to be invoked* for there to be a module instance created. Without the execution of the outer function, the creation of the inner scope and the closures would not occur.

首先，`CoolModule()`只是一个方法，但是*随着他*的执行他创建了一个模块实例创建了。如果不执行外部的方法，创建的内部作用域和闭包将不会被执行。

Secondly, the `CoolModule()` function returns an object, denoted by the object-literal syntax `{ key: value, ... }`. The object we return has references on it to our inner functions, but *not* to our inner data variables. We keep those hidden and private. It's appropriate to think of this object return value as essentially a **public API for our module**.

其次，`CoolModule()`方法返回一个对象，通过对象字面语法`{ key: value, ... }`被标记。我们返回的对象有我们内部方法的引用，但是*没有*我们的内部数据变量。我们保持他们的隐藏和私有。对我们返回的对象合适的理解是本质上他们是一个**我们模块的公开API**。

This object return value is ultimately assigned to the outer variable `foo`, and then we can access those property methods on the API, like `foo.doSomething()`.

返回的对象最终赋值给外部的变量`foo`，接下来我们可以读取他的API属性方法，像`foo.doSomething()`。

**Note:** It is not required that we return an actual object (literal) from our module. We could just return back an inner function directly. jQuery is actually a good example of this. The `jQuery` and `$` identifiers are the public API for the jQuery "module", but they are, themselves, just a function (which can itself have properties, since all functions are objects).

**注意：** 从我们模块里返回一个对象(字面上)这不是必须的。我们可以直接返回一个内部方法。jQuery事实上就是一个关于这个很好的例子。`jQuery` 和 `$` 标识符是jQuery"模块"的的公开API,但是他自己只是一个方法(他可以有自己的属性，因为所有的方法就是对象)。

The `doSomething()` and `doAnother()` functions have closure over the inner scope of the module "instance" (arrived at by actually invoking `CoolModule()`). When we transport those functions outside of the lexical scope, by way of property references on the object we return, we have now set up a condition by which closure can be observed and exercised.

`doSomething()` 和 `doAnother()`方法模块"实例"内部作用域的闭包(真正运行`CoolModule()`时到来)。当我们将这些方法通过引用我们返回的对象属性的方式搬移出词法作用域的外面，我们现在就设置好了闭包可以被观察和使用的条件。

To state it more simply, there are two "requirements" for the module pattern to be exercised:

更简单的说，模块模式被使用有两个"必要条件"

1. There must be an outer enclosing function, and it must be invoked at least once (each time creates a new module instance).

1. 必须有一个外部包围的方法，至少需要被执行一次(每次创建一个新的模块实例)。

2. The enclosing function must return back at least one inner function, so that this inner function has closure over the private scope, and can access and/or modify that private state.

2. 包围的方法需要返回至少一个内部方法，使这个内部方法有私有作用域的闭包，这样他可以访问和/或修改那个私有状态。

An object with a function property on it alone is not *really* a module. An object which is returned from a function invocation which only has data properties on it and no closured functions is not *really* a module, in the observable sense.

一个对象仅仅有一个方法属性不是*真正*的一个模块。一个对象通过方法执行返回而这个方法只有数据属性没有闭包方法这不是*真正*的一个模块，从可观察的角度来说。

The code snippet above shows a standalone module creator called `CoolModule()` which can be invoked any number of times, each time creating a new module instance. A slight variation on this pattern is when you only care to have one instance, a "singleton" of sorts:

代码片段关于一个独立的模块创建者叫做`CoolModule()`它可以被执行任意次数，每次执行创建一个新的模块实例。这个模式的一个轻微的变化是当你只想有一个实例，类似一个"单例"：

```js
var foo = (function CoolModule() {
	var something = "cool";
	var another = [1, 2, 3];

	function doSomething() {
		console.log( something );
	}

	function doAnother() {
		console.log( another.join( " ! " ) );
	}

	return {
		doSomething: doSomething,
		doAnother: doAnother
	};
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```

Here, we turned our module function into an IIFE (see Chapter 3), and we *immediately* invoked it and assigned its return value directly to our single module instance identifier `foo`.

这里，我们将我们的模块方法转移到一个IIFE钟(见第三章)，然后我们*立刻*执行他然后将返回的值直接赋值给我们的单独的模块实例标识符`foo`。

Modules are just functions, so they can receive parameters:

模块就是方法，所以它可以接受参数：

```js
function CoolModule(id) {
	function identify() {
		console.log( id );
	}

	return {
		identify: identify
	};
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```

Another slight but powerful variation on the module pattern is to name the object you are returning as your public API:

另外一个对于模块模式的轻微但是很强大的变化是将你返回的对象取名作为你的公开API:

```js
var foo = (function CoolModule(id) {
	function change() {
		// modifying the public API
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log( id );
	}

	function identify2() {
		console.log( id.toUpperCase() );
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

By retaining an inner reference to the public API object inside your module instance, you can modify that module instance **from the inside**, including adding and removing methods, properties, *and* changing their values.

通过保持一个在你模块实例内部的公开API对象的内部引用，你可以**从内部**修改模块实例，包括增加和删除方法，属性，*和* 修改他们的值。

### Modern Modules

### 现代模块

Various module dependency loaders/managers essentially wrap up this pattern of module definition into a friendly API. Rather than examine any one particular library, let me present a *very simple* proof of concept **for illustration purposes (only)**:

各种模块依赖的加载机制/管理机制本质上是包裹这个模块模式来定义一个友好的API。与其检验任何一个特别的库，**为了举例说明的目的(仅仅)** 让我来提出一个*非常简单*证明观点 ：

```js
var MyModules = (function Manager() {
	var modules = {};

	function define(name, deps, impl) {
		for (var i=0; i<deps.length; i++) {
			deps[i] = modules[deps[i]];
		}
		modules[name] = impl.apply( impl, deps );
	}

	function get(name) {
		return modules[name];
	}

	return {
		define: define,
		get: get
	};
})();
```

The key part of this code is `modules[name] = impl.apply(impl, deps)`. This is invoking the definition wrapper function for a module (passing in any dependencies), and storing the return value, the module's API, into an internal list of modules tracked by name.

这个代码的关键部分是`modules[name] = impl.apply(impl, deps)`。执行方法来替一个模块定义一个包围的方法(传入任何的依赖)，然后存储返回值，也就是模块的API，存储在一个模块内部的列表里用名字跟踪。

And here's how I might use it to define some modules:

这里是我如何使用来定一些模块：

```js
MyModules.define( "bar", [], function(){
	function hello(who) {
		return "Let me introduce: " + who;
	}

	return {
		hello: hello
	};
} );

MyModules.define( "foo", ["bar"], function(bar){
	var hungry = "hippo";

	function awesome() {
		console.log( bar.hello( hungry ).toUpperCase() );
	}

	return {
		awesome: awesome
	};
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
	bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

Both the "foo" and "bar" modules are defined with a function that returns a public API. "foo" even receives the instance of "bar" as a dependency parameter, and can use it accordingly.

"foo" 和 "bar"模块都通过一个方法来定义然后返回一个公开的API。 "foo"甚至可以接受"bar"的实例作为一个依赖参数，而且还可以相应的使用。

Spend some time examining these code snippets to fully understand the power of closures put to use for our own good purposes. The key take-away is that there's not really any particular "magic" to module managers. They fulfill both characteristics of the module pattern I listed above: invoking a function definition wrapper, and keeping its return value as the API for that module.

花费一些时间来测验这些代码片段来完全理解闭包的力量来为我们的好的目标使用。关键之处在于模块管理并不是任何特别的"魔法"。他实现了我列下来的关于模块模式的全部特性:执行一个方法定义包围者，然后保持他的返回值作为模块的API。

In other words, modules are just modules, even if you put a friendly wrapper tool on top of them.

换句话来说，模块只是模块，尽管你给他套上了友好的外衣。

### Future Modules

### 未来模块

ES6 adds first-class syntax support for the concept of modules. When loaded via the module system, ES6 treats a file as a separate module. Each module can both import other modules or specific API members, as well export their own public API members.

ES6加入了第一类型的语法来支持模块这个观念。当加载模块经由模块系统,ES6对待一个文件作为一个独立的模块。每个模块都可以引入其他模块或者具体的API对象，也可以输出自己的公开API对象。

**Note:** Function-based modules aren't a statically recognized pattern (something the compiler knows about), so their API semantics aren't considered until run-time. That is, you can actually modify a module's API during the run-time (see earlier `publicAPI` discussion).

**注意:** 方法为基础的模块不是一个静态被识别的模式(一些编译器知道的)，所以他们API语义不被考虑直到运行时。就因为这个，你可以在运行时修改一个模块的API(见更简单的`publicAPI`讨论)。

By contrast, ES6 Module APIs are static (the APIs don't change at run-time). Since the compiler knows *that*, it can (and does!) check during (file loading and) compilation that a reference to a member of an imported module's API *actually exists*. If the API reference doesn't exist, the compiler throws an "early" error at compile-time, rather than waiting for traditional dynamic run-time resolution (and errors, if any).

相反的，ES6模块API是静态的(API不会再运行时被修改)。因为编译器知道*那个*,他可以(和做)在(文件加载和)编译时检查一个被导入模块的引用对象API*是否存在*。如果API引用不存在，编译器会在编译时抛出一个"过早(early)"错误，而不是等到传统动态运行时解决方案(和错误,如果有的话)。

ES6 modules **do not** have an "inline" format, they must be defined in separate files (one per module). The browsers/engines have a default "module loader" (which is overridable, but that's well-beyond our discussion here) which synchronously loads a module file when it's imported.

ES6模块**不会**有一个"行内"格式化,他们必须在一个独立的文件中定义(每个一个模块)。浏览器/引擎有一个默认的"模块加载器"(可以被覆盖的，但是这个已经超出我们的在这里的讨论)他会在导入的时候同步在家一个模块。

Consider:

考虑：

**bar.js**
```js
function hello(who) {
	return "Let me introduce: " + who;
}

export hello;
```

**foo.js**
```js
// import only `hello()` from the "bar" module
import hello from "bar";

var hungry = "hippo";

function awesome() {
	console.log(
		hello( hungry ).toUpperCase()
	);
}

export awesome;
```

```js
// import the entire "foo" and "bar" modules
module foo from "foo";
module bar from "bar";

console.log(
	bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```

**Note:** Separate files **"foo.js"** and **"bar.js"** would need to be created, with the contents as shown in the first two snippets, respectively. Then, your program would load/import those modules to use them, as shown in the third snippet.

**注意:** **"foo.js"** 和 **"bar.js"** 这两个单独的文件需要各自被创建，他们的内容就是第一第二个片段。接下来，你的程序会加载/引入这些模块然后使用他们，就像第三段代码这样。

`import` imports one or more members from a module's API into the current scope, each to a bound variable (`hello` in our case). `module` imports an entire module API to a bound variable (`foo`, `bar` in our case). `export` exports an identifier (variable, function) to the public API for the current module. These operators can be used as many times in a module's definition as is necessary.

`import`从一个模块的API中导入一个或者多个对象到当前的作用域，每一个绑定一个变量(在我们的例子中是`hello`)。`module`导入一整个模块的API然后绑定一个变量(在我们例子中是`foo`, `bar`)。`export`输出一个标识符(变量,方法)作为当前模块的公开API。这些操作可以在必要的时候在一个模块的定义中使用多次。

The contents inside the *module file* are treated as if enclosed in a scope closure, just like with the function-closure modules seen earlier.

*模块文件* 内部的内容被当做包围的一个闭包作用域，就好像我们之前看到的方法闭包模块。

## Review (TL;DR)

## 回顾 (TL;DR)

Closure seems to the un-enlightened like a mystical world set apart inside of JavaScript which only the few bravest souls can reach. But it's actually just a standard and almost obvious fact of how we write code in a lexically scoped environment, where functions are values and can be passed around at will.

闭包似乎未被启发就像JavaScript中的一个神秘的世界只有一小部分最勇敢的灵魂可以达到。但是事实上是我们如何在一个词法作用域的环境中写代码的一个标准和几乎显而易见的事实，方法就是值可以根据你的需要被传来传去。

**Closure is when a function can remember and access its lexical scope even when it's invoked outside its lexical scope.**

**闭包就是当一个方法可以记住和读取自己的词法作用域即使当他在他的词法作用域外部被执行。**

Closures can trip us up, for instance with loops, if we're not careful to recognize them and how they work. But they are also an immensely powerful tool, enabling patterns like *modules* in their various forms.

如果我们不小心的识别闭包和不知道他们如何工作，在循环中使用实例，闭包可以将我们绊倒。但是他们也是一个有巨大力量的工具，启发了各种模式 *模块* 就是他的变化形式。

Modules require two key characteristics: 1) an outer wrapping function being invoked, to create the enclosing scope 2) the return value of the wrapping function must include reference to at least one inner function that then has closure over the private inner scope of the wrapper.

模块需要两个关键的特性：1)一个外部包围的方法需要被执行，来创建一个包围的作用域 2) 包围方法返回的值必须包含至少一个内部方法的引用,这个内部方法有包围函数的私有内部作用域的闭包。

Now we can see closures all around our existing code, and we have the ability to recognize and leverage them to our own benefit!

现在我们可以看到闭包在我们已经存在代码中到处都是，我们有能力去识别可以利用他们来为我们创造利益。
