---
layout: post
title: 'You Don''t Know JS: Scope & Closures-Chapter 1: What is Scope?'
date: 2017-07-29 12:15:10
tags:
---

# You Don't Know JS: Scope & Closures
# 你不知道的JS：作用域&闭包
# Chapter 1: What is Scope?
# 第一章：什么是作用域

One of the most fundamental paradigms of nearly all programming languages is the ability to store values in variables, and later retrieve or modify those values. In fact, the ability to store values and pull values out of variables is what gives a program *state*.

几乎所有的语言其中一个最基本的理论就是存储值到变量中,然后在去获取或者修改值.事实上,从变量中存值或者取值是给了程序*状态*


Without such a concept, a program could perform some tasks, but they would be extremely limited and not terribly interesting.

没有了这些概念,程序还是可以执行某些任务,但是会极度的受局限而且完全没有乐趣可言。

But the inclusion of variables into our program begets the most interesting questions we will now address: where do those variables *live*? In other words, where are they stored? And, most importantly, how does our program find them when it needs them?

但是一旦在程序中包含了变量这个概念就产生了很多有趣的问题我们需要对付:变量在哪里*存活*?换句话来说,它存在哪里?而却更重要的问题是,我们的程序是怎么在它需要使用变量的时候找到他的?

These questions speak to the need for a well-defined set of rules for storing variables in some location, and for finding those variables at a later time. We'll call that set of rules: *Scope*.

这些问题需要定义一些规则把变量存在某些位置,在稍后需要使用的时候把变量找出来.我们把这个规则叫做:*作用域*

But, where and how do these *Scope* rules get set?

但是,这些*规则*是如何以及在哪里存取变量的呢?

## Compiler Theory

## 编译理论

It may be self-evident, or it may be surprising, depending on your level of interaction with various languages, but despite the fact that JavaScript falls under the general category of "dynamic" or "interpreted" languages, it is in fact a compiled language. It is *not* compiled well in advance, as are many traditionally-compiled languages, nor are the results of compilation portable among various distributed systems.

这可能不言而喻,或者会令你吃惊,这都取决于你各种语言相互之间的水平,尽管大多数人叫JavaScript归为"动态语言"或者"解释语言",但是事实上他也是编译语言。他*不是*像传统的编译语言的方式预先被编译好,编译好的结果也不能再各种不同分布式系统中移植。

But, nevertheless, the JavaScript engine performs many of the same steps, albeit in more sophisticated ways than we may commonly be aware, of any traditional language-compiler.

但是,尽管如此,JavaScript引擎和大多数传统的编译语言一样执行相同的步骤,以我们不容易意识到的更精妙的方式进行编译。

In traditional compiled-language process, a chunk of source code, your program, will undergo typically three steps *before* it is executed, roughly called "compilation":

在传统的编译语言过程中,组成的源代码,你的程序,值执行*之前*需要经历三个典型的步骤,这些步骤大致可以叫做"编译"。

1. **Tokenizing/Lexing:** breaking up a string of characters into meaningful (to the language) chunks, called tokens. For instance, consider the program: `var a = 2;`. This program would likely be broken up into the following tokens: `var`, `a`, `=`, `2`, and `;`. Whitespace may or may not be persisted as a token, depending on whether it's meaningful or not.

1. **分词/词法分析:** 把字符串拆分成有意义的(对相应的计算机语言来说)字符块,叫做分词。举例来说,注意一下这段代码:`var a=2;`.这段代码基本上会被拆分成以下几个部分:`var`, `a`, `=`, `2`,还有 `;`.空白符或许会被认为是一个部分,这要取决于它对于这个语言来说是否有意义。

    **Note:** The difference between tokenizing and lexing is subtle and academic, but it centers on whether or not these tokens are identified in a *stateless* or *stateful* way. Put simply, if the tokenizer were to invoke stateful parsing rules to figure out whether `a` should be considered a distinct token or just part of another token, *that* would be **lexing**.

    **注意：** 分词和词法分析的区别是微妙和学术性的,但是这个主要和解析方式是根据*有状态*还是*无状态*的方式来进行的有关。简单来说,如果分词分析器借助有状态的解析规则去搞清楚 `a`是一个唯一的组成部分或者只是其他部分的组成部分,*这* 就叫做**词法分析**.(译者注:如果作为一个无状态的语言那么上文的a肯定是作为一个单独的部分用来被解析的)

2. **Parsing:** taking a stream (array) of tokens and turning it into a tree of nested elements, which collectively represent the grammatical structure of the program. This tree is called an "AST" (<b>A</b>bstract <b>S</b>yntax <b>T</b>ree).

2. **语法分析:** 取出分词完的流(数组)放入一个嵌套元素的树中,用来收集和代表程序的语法结构。这个树叫做"AST"(<b>A</b>bstract <b>S</b>yntax <b>T</b>ree).

    The tree for `var a = 2;` might start with a top-level node called `VariableDeclaration`, with a child node called `Identifier` (whose value is `a`), and another child called `AssignmentExpression` which itself has a child called `NumericLiteral` (whose value is `2`).

    `var a = 2;`的树最上层的节点叫做`变量声明`,他的子节点叫做`标识符`(他的值是`a`),他的另外一个子节点叫做`分配表达式`他又一个节点叫做`数字字面量`(他的值是`2`)。

3. **Code-Generation:** the process of taking an AST and turning it into executable code. This part varies greatly depending on the language, the platform it's targeting, etc.

3. **生成代码:** 这个过程是把AST转变成可执行的代码。这个部分的变化跟他的语言,跟他的目标平台等等有很大的关系。

    So, rather than get mired in details, we'll just handwave and say that there's a way to take our above described AST for `var a = 2;` and turn it into a set of machine instructions to actually *create* a variable called `a` (including reserving memory, etc.), and then store a value into `a`.

    然而,相比在细节中挣扎,我们可以用简单的方式去描述它, `var a = 2;` 的AST转换成机器层面的代码的方式事实上就是*创建*了一个变量叫做`a`(包括开辟了内存,等等)，然后在把值存入到`a`中。


    **Note:** The details of how the engine manages system resources are deeper than we will dig, so we'll just take it for granted that the engine is able to create and store variables as needed.

    **注意:** 引擎如何管理系统资源的细节比我们挖掘的要深,我们只需要知道引擎有能力在它需要的时候创建和存储变量。

The JavaScript engine is vastly more complex than *just* those three steps, as are most other language compilers. For instance, in the process of parsing and code-generation, there are certainly steps to optimize the performance of the execution, including collapsing redundant elements, etc.

如同其他的语言编译器一样JavaScript引擎编译步骤比这*区区*三部要复杂很多。举例来说,在语法分析和生成代码的过程中,肯定有一些步骤来优化执行效率,包括折叠修剪节点,等等。

So, I'm painting only with broad strokes here. But I think you'll see shortly why *these* details we *do* cover, even at a high level, are relevant.

所以,这里我只能简单的描绘一下.但是我想你很快可以看的出来，为什么*这些*细节我们需要知道,即使是在高的层面上,他们也是有意义的。

For one thing, JavaScript engines don't get the luxury (like other language compilers) of having plenty of time to optimize, because JavaScript compilation doesn't happen in a build step ahead of time, as with other languages.

首先,JavaScript引擎没有很多(和其他语言编译器一样)充裕的时间去做优化,因为就其他语言而言JavaScript编译器没有在一开始构建这个步骤。

For JavaScript, the compilation that occurs happens, in many cases, mere microseconds (or less!) before the code is executed. To ensure the fastest performance, JS engines use all kinds of tricks (like JITs, which lazy compile and even hot re-compile, etc.) which are well beyond the "scope" of our discussion here.

对JavaScript而言,大多数情况下,编译才刚发生,仅仅几毫秒(甚至更少)后代码就被执行了。为了确保最快的效率,JS用了各种各样的技巧(像JITs,懒编译和甚至热重编译,等等)这些都超出了我们的在这里讨论的"作用域"的范围之外。

Let's just say, for simplicity's sake, that any snippet of JavaScript has to be compiled before (usually *right* before!) it's executed. So, the JS compiler will take the program `var a = 2;` and compile it *first*, and then be ready to execute it, usually right away.

为了简单起见,我们可以说,任何一小段JavaScript执行之前(通常是非常快的)都被编译了。所以，JS编译器会把代码`var a = 2;`首先编译好,然后准备执行,通常是立即执行了。



## Understanding Scope

## 理解作用域

The way we will approach learning about scope is to think of the process in terms of a conversation. But, *who* is having the conversation?

我们学习作用域的方式是把过程想象成是一种对话的表达方式。但是,谁在对话呢?

### The Cast

### 演员表

Let's meet the cast of characters that interact to process the program `var a = 2;`, so we understand their conversations that we'll listen in on shortly:

现在让我们认识一下处理程序`var a = 2;`过程中的几个重要演员,这样我们就可以理解他们接下来马上要进行的对话了。

1. *Engine*: responsible for start-to-finish compilation and execution of our JavaScript program.

1. *引擎*:负责从头到尾的编译和执行我们的JavaScript程序。

2. *Compiler*: one of *Engine*'s friends; handles all the dirty work of parsing and code-generation (see previous section).

2. *编译器*：引擎的朋友;负责语法分析和生成代码中(见之前的段落)的所有脏活累活

3. *Scope*: another friend of *Engine*; collects and maintains a look-up list of all the declared identifiers (variables), and enforces a strict set of rules as to how these are accessible to currently executing code.

3. *作用域*: 引擎的另外一位朋友；收集和维护已申明标识符(变量)的列表,然后为当前正在执行的代码强行制定一个严格的规则来规定它们(标识符列表)哪些是可访问的。

For you to *fully understand* how JavaScript works, you need to begin to *think* like *Engine* (and friends) think, ask the questions they ask, and answer those questions the same.

为了让你*全面理解*JavaScript是如何工作的,你需要开始*想*引擎(和他的朋友)所想,问引擎所问,然后像引擎一样去回答。

### Back & Forth

### 前前后后

When you see the program `var a = 2;`, you most likely think of that as one statement. But that's not how our new friend *Engine* sees it. In fact, *Engine* sees two distinct statements, one which *Compiler* will handle during compilation, and one which *Engine* will handle during execution.

当你看到程序`var a = 2;`,你大概会想这是一个声明。但是这不是我们新朋友*引擎*看到的。事实上,引擎看到两个有区别的表达式.一个是*编译器*在编译期间会处理的,一个是*引擎*在执行期间会处理的。

So, let's break down how *Engine* and friends will approach the program `var a = 2;`.

好让我们把*引擎*和他朋友如何处理程序`var a = 2;`的过程划分来看。

The first thing *Compiler* will do with this program is perform lexing to break it down into tokens, which it will then parse into a tree. But when *Compiler* gets to code-generation, it will treat this program somewhat differently than perhaps assumed.

首先在这段代码上要做的事是*编译器*会对它进行词法分析把他拆分到每个部分中去,之后会被分析到树种去。但是当*编译器*到代码生成的阶段,它会以某种和假定方式不同的方式来对待这段程序。

A reasonable assumption would be that *Compiler* will produce code that could be summed up by this pseudo-code: "Allocate memory for a variable, label it `a`, then stick the value `2` into that variable." Unfortunately, that's not quite accurate.

*编译器*会如何处理代码的一个合理假设可以用一段伪代码来概括:"给变量分配内存,标记为`a`,然后把值`2`插入到变量中."不幸的是,这并不准确。

*Compiler* will instead proceed as:

编译器其实会经过下面几个过程:

1. Encountering `var a`, *Compiler* asks *Scope* to see if a variable `a` already exists for that particular scope collection. If so, *Compiler* ignores this declaration and moves on. Otherwise, *Compiler* asks *Scope* to declare a new variable called `a` for that scope collection.

1. 遭遇 `var a`,编译器告诉作用域去看一下变量 `a`是否已经存在于特指的作用域集合中。如果存在,编译器会忽略这个申明然后继续往下。否则的话,编译器会告诉作用域在作用域的的集合中声明一个新的变量叫做`a`。

2. *Compiler* then produces code for *Engine* to later execute, to handle the `a = 2` assignment. The code *Engine* runs will first ask *Scope* if there is a variable called `a` accessible in the current scope collection. If so, *Engine* uses that variable. If not, *Engine* looks *elsewhere* (see nested *Scope* section below).

2.编译器然后为引擎接下的执行生成代码,处理`a = 2`表达式。代码引擎启动会首先告诉作用域查看是否有可访问的变量叫做 `a`存在于当前订单作用域集合列表中。如果存在引擎会使用这个变量。如果不存在,引擎会查看*其它地方*(见接下来的嵌套*作用域*)。

If *Engine* eventually finds a variable, it assigns the value `2` to it. If not, *Engine* will raise its hand and yell out an error!

如果引擎最终找到了变量,他会分配值 `2`给他。如果没有,引擎会举起手然后大声的喊出一个错误。

To summarize: two distinct actions are taken for a variable assignment: First, *Compiler* declares a variable (if not previously declared in the current scope), and second, when executing, *Engine* looks up the variable in *Scope* and assigns to it, if found.

总结一下:两个有区别的操作来处理一个分配变量:首先,编译器会声明变量(如果之前没有在当前的作用域中声明过的话)，然后第二步,会开始执行,引擎会在作用域中查找变量然后分配值,找得到的前提下。

### Compiler Speak
### 编译器说

We need a little bit more compiler terminology to proceed further with understanding.

为了更远的过程我们需要了解一些编译器的专业术语。

When *Engine* executes the code that *Compiler* produced for step (2), it has to look-up the variable `a` to see if it has been declared, and this look-up is consulting *Scope*. But the type of look-up *Engine* performs affects the outcome of the look-up.

当引擎执行编译器在第二步生成的代码时,他会查看变量`a` 如果他已经背声明了,这次查看是咨询作用域的。但是引擎的处理查询类型会影响查询的结果。

In our case, it is said that *Engine* would be performing an "LHS" look-up for the variable `a`. The other type of look-up is called "RHS".

在我们的例子中,我们说引擎会执行一个"LHS"查询变量 `a`.另外一个类型的查询叫做"RHS"。

I bet you can guess what the "L" and "R" mean. These terms stand for "Left-hand Side" and "Right-hand Side".

我打赌你已经猜到了"L"和"R"的意思。他们分别代表了"Left-hand Side"(左边) 和 "Right-hand Side"(右边边).

Side... of what? **Of an assignment operation.**

边... 哪个边? **就是赋值操作** (赋值操作的左边和右边)

In other words, an LHS look-up is done when a variable appears on the left-hand side of an assignment operation, and an RHS look-up is done when a variable appears on the right-hand side of an assignment operation.

换句话来说,LHS查询是当变量出现在赋值操作的左边,RHS查询是当变量出现在赋值操作的右边。

Actually, let's be a little more precise. An RHS look-up is indistinguishable, for our purposes, from simply a look-up of the value of some variable, whereas the LHS look-up is trying to find the variable container itself, so that it can assign. In this way, RHS doesn't *really* mean "right-hand side of an assignment" per se, it just, more accurately, means "not left-hand side".

让我们来描述的更准确一点。对我们的意图来说,RHS查询是不易察觉的,他是简单的查询一个变量的值,然而LHS查询是尝试找到变量去装载他自己,然后进行赋值。在这种情况下,RHS不是真正意义上我们之前提到的的“右边赋值(right-hand side of an assignment)”,更准确来说,他的意思是"不是左边(not left-hand side)"。

Being slightly glib for a moment, you could also think "RHS" instead means "retrieve his/her source (value)", implying that RHS means "go get the value of...".

在这番油腔滑调之后,你也可以认为"RHS"是"retrieve his/her source (value)"的简写,暗示着"去获得....的值"。

Let's dig into that deeper.

让我们更深入一点。

When I say:

当我说:

```js
console.log( a );
```

The reference to `a` is an RHS reference, because nothing is being assigned to `a` here. Instead, we're looking-up to retrieve the value of `a`, so that the value can be passed to `console.log(..)`.

对`a`的引用是一个RHS引用,因为没有给`a`赋值。相反的,他只是取得 `a`的值,然后这个值就可以被传入`console.log(..)`中。

By contrast:

相比之下

```js
a = 2;
```

The reference to `a` here is an LHS reference, because we don't actually care what the current value is, we simply want to find the variable as a target for the `= 2` assignment operation.

这里对 `a`的引用是一个LHS引用,因为我们并不真正关心当前的值,我们简单的想找到变量作为目标进行`= 2` 的赋值操作。

**Note:** LHS and RHS meaning "left/right-hand side of an assignment" doesn't necessarily literally mean "left/right side of the `=` assignment operator". There are several other ways that assignments happen, and so it's better to conceptually think about it as: "who's the target of the assignment (LHS)" and "who's the source of the assignment (RHS)".

**注意** LHS和RHS的意思"赋值的左边和右边"并不是必须像字面意思"`=`赋值操作的左边和右边"这样理解。

Consider this program, which has both LHS and RHS references:

考虑一下这个程序,哪个即使LHS引用又是RHS引用:

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

The last line that invokes `foo(..)` as a function call requires an RHS reference to `foo`, meaning, "go look-up the value of `foo`, and give it to me." Moreover, `(..)` means the value of `foo` should be executed, so it'd better actually be a function!

最后一行执行`foo(..)`作为一个方法调用为 `foo`执行了一个RHS引用,意思是,"去看一下`foo`的值,然后把值给我。"此外，`(..)`意味着`foo`的值是执行之后的，所以它最好实际上是一个方法。

There's a subtle but important assignment here. **Did you spot it?**

这里有一个不易察觉但是很重要的赋值。**你注意到了么?**

You may have missed the implied `a = 2` in this code snippet. It happens when the value `2` is passed as an argument to the `foo(..)` function, in which case the `2` value is **assigned** to the parameter `a`. To (implicitly) assign to parameter `a`, an LHS look-up is performed.

你也许忽略了暗示着 `a = 2`的这小段代码。他发生在当值`2`作为参数传入到`foo(..)`这个方法中,在这种情况下`2` 这个值**被赋值**给了参数 `a`。给(暗示的)`a`参数赋值,一个LHS查看就被执行了

There's also an RHS reference for the value of `a`, and that resulting value is passed to `console.log(..)`. `console.log(..)` needs a reference to execute. It's an RHS look-up for the `console` object, then a property-resolution occurs to see if it has a method called `log`.

他们也做了RHS引用来获取 `a`的值,然后将结果传入到 `console.log(..)`。 `console.log(..)`需要一个引用来执行。这里对`console`进行一个RHS查看.然后属性解析被执行来查看是否有一个方法叫做`log`。

Finally, we can conceptualize that there's an LHS/RHS exchange of passing the value `2` (by way of variable `a`'s RHS look-up) into `log(..)`. Inside of the native implementation of `log(..)`, we can assume it has parameters, the first of which (perhaps called `arg1`) has an LHS reference look-up, before assigning `2` to it.

最后,我们可以将概念化,这里有一个LHS/RHS交换将`2`(顺便说一下`a`变量是一个RHS查看)的值传入到`log(..)`中。在原生方法`log(..)`里面,我们可以假设他有参数,第一个参数(或许叫做`arg1`)发生了一个LHS引用查询将`2`赋值给了它。

**Note:** You might be tempted to conceptualize the function declaration `function foo(a) {...` as a normal variable declaration and assignment, such as `var foo` and `foo = function(a){...`. In so doing, it would be tempting to think of this function declaration as involving an LHS look-up.

**注意:** 你可能被诱导将这里的方法声明`function foo(a) {...`认为是普通的变量声明和赋值,就像`var foo`和`foo = function(a){...`那样，这可能会诱使你觉得方法声明是涉及了一次LHS查看。

However, the subtle but important difference is that *Compiler* handles both the declaration and the value definition during code-generation, such that when *Engine* is executing code, there's no processing necessary to "assign" a function value to `foo`. Thus, it's not really appropriate to think of a function declaration as an LHS look-up assignment in the way we're discussing them here.

然而,很微妙但是很重要的区别就是*编译器*同时处理代码生成阶段值的声明和的定义,当*引擎*是执行代码,没有过程必须要将方法值"赋值"给 `foo`。因此，在我们这里的讨论将方法声明当做是一次LHS赋值查看不太合适。

### Engine/Scope Conversation

### 引擎/作用域 交谈

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

Let's imagine the above exchange (which processes this code snippet) as a conversation. The conversation would go a little something like this:

让我们想象一下将交换(这小段代码的过程)看做是一场交谈。这段交谈会像下面一样展开：

> ***Engine***: Hey *Scope*, I have an RHS reference for `foo`. Ever heard of it?

> ***引擎***: Hey *作用域*, 我要为`foo`做一次RHS引用,你听说过他么?

> ***Scope***: Why yes, I have. *Compiler* declared it just a second ago. He's a function. Here you go.

> ***作用域***: 怎么了 是的, 我这里有. *编译器* 刚刚声明了. 它是一个方法. 给你吧.

> ***Engine***: Great, thanks! OK, I'm executing `foo`.

> ***Engine***: 太好了, 谢谢! OK, 我正在执行 `foo`了.

> ***Engine***: Hey, *Scope*, I've got an LHS reference for `a`, ever heard of it?

> ***引擎***: Hey, *作用域*, 我要为`a`做一次LHS引用, 你听说过他么?

> ***Scope***: Why yes, I have. *Compiler* declared it as a formal parameter to `foo` just recently. Here you go.

> ***作用域***: 怎么了 是的,我又. *编译器* 刚刚申明了它作为`foo`的参数。拿去吧。

> ***Engine***: Helpful as always, *Scope*. Thanks again. Now, time to assign `2` to `a`.

> ***引擎***: 你一直那么乐于助人, *作用域*. 再次感谢. 现在是把 `2`赋值给`a`.

> ***Engine***: Hey, *Scope*, sorry to bother you again. I need an RHS look-up for `console`. Ever heard of it?

> ***引擎***: Hey, *作用域*, 抱歉又打扰你。我需要为`console`做一次RHS查看。你听说过他么？

> ***Scope***: No problem, *Engine*, this is what I do all day. Yes, I've got `console`. He's built-in. Here ya go.

> ***作用域***: 没问题, *Engine*, 我是我整天都在干的事情。是的,我这里有`console`。他是构建好的.给你吧。

> ***Engine***: Perfect. Looking up `log(..)`. OK, great, it's a function.

> ***引擎***: 太棒了. 查看一下 `log(..)`. OK, 好极了, 他是一个方法.

> ***Engine***: Yo, *Scope*. Can you help me out with an RHS reference to `a`. I think I remember it, but just want to double-check.

> ***引擎***: Yo, *作用域*. 可以帮我取出`a`我要为他做一次RHS引用. 我想我记得他,但是我想要在确认一下。

> ***Scope***: You're right, *Engine*. Same guy, hasn't changed. Here ya go.

> ***作用域***: 你说的没错, *引擎*. 还是他, 没变. 给你吧.

> ***Engine***: Cool. Passing the value of `a`, which is `2`, into `log(..)`.

> ***Engine***: Cool. 把值 `2`给`a`, 传入到 `log(..)`中去.

> ...

### Quiz

### 小测验

Check your understanding so far. Make sure to play the part of *Engine* and have a "conversation" with the *Scope*:

检查一下你是否理解了。确保你在扮演*Engine*然后和*作用域*：

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

1. 找出所有所有的LHS查看(总共有3个!)。

2. Identify all the RHS look-ups (there are 4!).

2. 找出所有订单RHS查看(总共有4个!)。

**Note:** See the chapter review for the quiz answers!

**注意:** 答案见章节回顾！

## Nested Scope

## 嵌套作用域

We said that *Scope* is a set of rules for looking up variables by their identifier name. There's usually more than one *Scope* to consider, however.

我们说*作用域*是设置一个规则根据识别的名称来找寻变量。然而经常有超过一个*作用域*我们需要考虑。

Just as a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, *Engine* consults the next outer containing scope, continuing until found or until the outermost (aka, global) scope has been reached.

就像块或者方法可以被嵌套在另外一个块或者方法中,作用域也是嵌套被嵌套在其他作用域里的。所以,如果变量不能再当前作用域中被找到，引擎会咨询包含他的下一个作用域,一直到找到或者到达最外层(全局)的作用域为止。

Consider:

仔细考虑:

```js
function foo(a) {
	console.log( a + b );
}

var b = 2;

foo( 2 ); // 4
```

The RHS reference for `b` cannot be resolved inside the function `foo`, but it can be resolved in the *Scope* surrounding it (in this case, the global).

这个为 `b`做的RHS引用不能再方法`foo`中被解决,但是可以在包围他的作用域中被解决(在这个例子中是全局)。

So, revisiting the conversations between *Engine* and *Scope*, we'd overhear:

所以,在看一下引擎和作用域的对话,我们会听到:

> ***Engine***: "Hey, *Scope* of `foo`, ever heard of `b`? Got an RHS reference for it."

> ***引擎***: "Hey, `foo`*作用域*, 你听说过 `b` 么? 我要给他做一次RHS引用。"

> ***Scope***: "Nope, never heard of it. Go fish."

> ***作用域***: "没有, 从没听过. 去找别人吧."

> ***Engine***: "Hey, *Scope* outside of `foo`, oh you're the global *Scope*, ok cool. Ever heard of `b`? Got an RHS reference for it."

> ***引擎***: "Hey, `foo`外层的 *作用域*, 噢 你是全局 *作用域*. 听说过 `b` 么? 我要给他做一次RHS引用。"

> ***Scope***: "Yep, sure have. Here ya go."

> ***作用域***: "是的, 当然有. 拿去吧."

The simple rules for traversing nested *Scope*: *Engine* starts at the currently executing *Scope*, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

横穿嵌套*作用域*的简单规则:引擎从当前正在执行的作用域开始查找变量,如果没有找到,继续往上一层找,然后一直下去。如果到达最外层的全局作用域,不管有没有找到这次搜索结束。

### Building on Metaphors

### 建筑比喻

To visualize the process of nested *Scope* resolution, I want you to think of this tall building.

为了使嵌套作用域的解决形成思维图像,我想你能思考一下这个高的建筑。

<img src="fig1.png" width="250">

The building represents our program's nested *Scope* rule set. The first floor of the building represents your currently executing *Scope*, wherever you are. The top level of the building is the global *Scope*.

这个建筑代表我们程序的嵌套作用域设置规则。不论你在什么位置,建筑物的第一层代表我们当前执行的作用域。建筑物的最高层就是全局作用域。

You resolve LHS and RHS references by looking on your current floor, and if you don't find it, taking the elevator to the next floor, looking there, then the next, and so on. Once you get to the top floor (the global *Scope*), you either find what you're looking for, or you don't. But you have to stop regardless.

通过观察你的当前楼层来解决你的LHS和RHS引用，如果你没有找到,乘坐电梯到下一层,看看那边,然后一层层往上找。一旦当你到达顶层(全局作用域),可能你已经找到你想找的,又或者没有找到。但是不管怎么样你都得停下来。

## Errors

## 错误

Why does it matter whether we call it LHS or RHS?

为什么分清LHS或者RHS很重要?

Because these two types of look-ups behave differently in the circumstance where the variable has not yet been declared (is not found in any consulted *Scope*).

因为这两个类型的查找在当变量没有在环境中声明的情况下(没有在任何一个被咨询的作用中找到)有着不一样的表现。

Consider:

考虑一下

```js
function foo(a) {
	console.log( a + b );
	b = a;
}

foo( 2 );
```

When the RHS look-up occurs for `b` the first time, it will not be found. This is said to be an "undeclared" variable, because it is not found in the scope.

当一开始对`b`执行RHS查找,它将会找不到。这被叫做一个"未声明"的变量,因为作用域中不能被找到。

If an RHS look-up fails to ever find a variable, anywhere in the nested *Scope*s, this results in a `ReferenceError` being thrown by the *Engine*. It's important to note that the error is of the type `ReferenceError`.

如果对一个变量进行RHS查找,不论哪个嵌套作用中都找不到，*引擎*会抛出一个'引用错误（ReferenceError）'的结果。注意这个错误是一个类型为'引用错误（ReferenceError）'这很重要。

By contrast, if the *Engine* is performing an LHS look-up and arrives at the top floor (global *Scope*) without finding it, and if the program is not running in "Strict Mode" [^note-strictmode], then the global *Scope* will create a new variable of that name **in the global scope**, and hand it back to *Engine*

相比之下,如果*Engine*执行一个LHS查找然后一直到达顶层(全局作用域)都没有找到,如果程序不是运行在"严格模式(Strict Mode)"[^note-strictmode]下,这样全局作用域就会在全局作用域里用这个名字创建一个新的变量,然后把执行权交还给引擎。

*"No, there wasn't one before, but I was helpful and created one for you."*

"不，这个之前没有,但是我会帮忙替你创造一个。"

"Strict Mode" [^note-strictmode], which was added in ES5, has a number of different behaviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global *Scope*'d variable to hand back from an LHS look-up, and *Engine* would throw a `ReferenceError` similarly to the RHS case.

"严格模式(Strict Mode)"[^note-strictmode],在ES5中新增,和普通/宽松/懒惰模式有几个不同的行为。其中一个行为就是不允许 自动/不声明 的创建全局变量。在这种情况下,全局作用域的变量将不会交给LHS查看,然后引擎会抛出一个`引用错误(ReferenceError)`类似RHS的情况。

Now, if a variable is found for an RHS look-up, but you try to do something with its value that is impossible, such as trying to execute-as-function a non-function value, or reference a property on a `null` or `undefined` value, then *Engine* throws a different kind of error, called a `TypeError`.

现在,如果一个变量通过一个RHS查看找到,但是你尝试对他的值做一些做不到的事,类似尝试像调用方法一样调用一个不是方法的值,或者引用一个`null` 或者 `undefined`的值,然后引擎会抛出一个不一样的错误,叫做`TypeError(类型错误)`。

`ReferenceError` is *Scope* resolution-failure related, whereas `TypeError` implies that *Scope* resolution was successful, but that there was an illegal/impossible action attempted against the result.

`ReferenceError(引用错误)`作用域解决失败有关,然而`TypeError`暗示着作用域解决成功,但是尝试对结果做 不合法/不可能 的操作。


## Review (TL;DR)

## 回顾

Scope is the set of rules that determines where and how a variable (identifier) can be looked-up. This look-up may be for the purposes of assigning to the variable, which is an LHS (left-hand-side) reference, or it may be for the purposes of retrieving its value, which is an RHS (right-hand-side) reference.

作用域是设置一个规则来查明在哪里和如何查看一个变量(鉴别器)。查看可能是以给变量赋值为目的,这个就是LHS(左手(left-hand-side))引用,或者是以获得值为目的,这个就是RHS(右边(right-hand-side))引用。

LHS references result from assignment operations. *Scope*-related assignments can occur either with the `=` operator or by passing arguments to (assign to) function parameters.

LHS引用从赋值操作获得结果。作用域相关的赋值操作可能会在`=`操作符或者传入(赋值)给方法的参数中发生。

The JavaScript *Engine* first compiles code before it executes, and in so doing, it splits up statements like `var a = 2;` into two separate steps:

JavaScript引擎会在执行代码前首次编译代码，因此，他会把`var a = 2;`进行拆分进行两个不同的步骤:

1. First, `var a` to declare it in that *Scope*. This is performed at the beginning, before code execution.

1. 首先,`var a`会在他的作用域中声明。这个会代码执行前在一开始就被执行。

2. Later, `a = 2` to look up the variable (LHS reference) and assign to it if found.

2. 然后, `a = 2` 查看变量如果存在的话(LHS引用)然后赋值。

Both LHS and RHS reference look-ups start at the currently executing *Scope*, and if need be (that is, they don't find what they're looking for there), they work their way up the nested *Scope*, one scope (floor) at a time, looking for the identifier, until they get to the global (top floor) and stop, and either find it, or don't.

LHS和RHS引用都是从当前执行的作用域开始查找,然后如果需要等待话(他们没有在那个作用域中找到他们想要的东西),他们会继续在嵌套的作用域中查找,一次只在一个作用域(层)找,不论找到还是没找到,只要达到全局(最高层)就停止。

Unfulfilled RHS references result in `ReferenceError`s being thrown. Unfulfilled LHS references result in an automatic, implicitly-created global of that name (if not in "Strict Mode" [^note-strictmode]), or a `ReferenceError` (if in "Strict Mode" [^note-strictmode]).

未成功的RHS引用结果是抛出一个`ReferenceError`。未成功的LHS引用结果会自动,隐藏的用这个名字创建一个全局变量(如果不是严格模式下"Strict Mode" [^note-strictmode]),或者抛出`ReferenceError`(如果在严格模式下"Strict Mode" [^note-strictmode])。

### Quiz Answers

### 测验答案

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

	**`c = ..`, `a = 2` (implicit param assignment) and `b = ..`**

2. Identify all the RHS look-ups (there are 4!).

    **`foo(2..`, `= a;`, `a + ..` and `.. + b`**


[^note-strictmode]: MDN: [Strict Mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode)
