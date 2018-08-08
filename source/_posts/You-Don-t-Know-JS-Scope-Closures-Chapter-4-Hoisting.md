---
layout: post
title: 'You Don''t Know JS: Scope & Closures-Chapter 4: Hoisting'
date: 2017-07-29 12:16:43
tags:
---

# You Don't Know JS: Scope & Closures
# Chapter 4: Hoisting

# 你不知道的JS：作用域和闭包
# 第四章：提升

By now, you should be fairly comfortable with the idea of scope, and how variables are attached to different levels of scope depending on where and how they are declared. Both function scope and block scope behave by the same rules in this regard: any variable declared within a scope is attached to that scope.

现在,你应该对作用域的概念有了相当的熟悉，变量如何依附于不同层级的作用域取决于他们在哪里和如何声明。方法作用域和块作用都有相同的规则：任何变量在某个作用域中声明他就依附于这个作用域。

But there's a subtle detail of how scope attachment works with declarations that appear in various locations within a scope, and that detail is what we will examine here.

但是有一个微妙的细节就是作用域和似乎在作用域内部各种位置变量是如何工作的，这个细节是我们将会在这里调查的。

## Chicken Or The Egg?

## 鸡或者是蛋

There's a temptation to think that all of the code you see in a JavaScript program is interpreted line-by-line, top-down in order, as the program executes. While that is substantially true, there's one part of that assumption which can lead to incorrect thinking about your program.

自上往下的顺序,在程序执行的时候，会诱使你觉得你看到的JavaScript程序是一行接一行被解释的。虽然这大体上是正确的，但是这个猜测的一部分会使你不准确的想你的程序。

Consider this code:

考虑这个代码：

```js
a = 2;

var a;

console.log( a );
```

What do you expect to be printed in the `console.log(..)` statement?

`console.log(..)`表达式会输出什么你是如何猜测的?

Many developers would expect `undefined`, since the `var a` statement comes after the `a = 2`, and it would seem natural to assume that the variable is re-defined, and thus assigned the default `undefined`. However, the output will be `2`.

很多开发者会猜测`undefined`,因为`var a`表达式在`a = 2`之后,这很自然的会假设变量被重复定义了，这样被赋值默认值`undefined`。然而,输出将会是`2`。

Consider another piece of code:

考虑一下另外一段代码:

```js
console.log( a );

var a = 2;
```

You might be tempted to assume that, since the previous snippet exhibited some less-than-top-down looking behavior, perhaps in this snippet, `2` will also be printed. Others may think that since the `a` variable is used before it is declared, this must result in a `ReferenceError` being thrown.

因为前面一段代码展示的绝不是自上到下的的举止，你也许会被诱惑去假设，也许在这段片段中，仍然将输出`2`。其他可能会像因为  `a`在声明之前被使用，结果肯定会抛出一个`ReferenceError`(引用错误)的错误。

Unfortunately, both guesses are incorrect. `undefined` is the output.

不幸的是，他们的猜测都是不正确的。 `undefined` 才是输出。

**So, what's going on here?** It would appear we have a chicken-and-the-egg question. Which comes first, the declaration ("egg"), or the assignment ("chicken")?

**所以，这里发生了什么？** 这似乎看起来让我们有了一个先有鸡还是先有蛋的问题。什么在前，是声明("蛋")，或者是赋值("鸡")。

## The Compiler Strikes Again

## 编译者再次来袭

To answer this question, we need to refer back to Chapter 1, and our discussion of compilers. Recall that the *Engine* actually will compile your JavaScript code before it interprets it. Part of the compilation phase was to find and associate all declarations with their appropriate scopes. Chapter 2 showed us that this is the heart of Lexical Scope.

为了回答这个问题，我们需要参考之前的第一章，我们讨论过的编译器。回顾之前*引擎*实际上会在解释之前编译你的JavaScript代码。编译阶段的有一部分就是找到所有变量然后将他们和所占用的作用域联系起来。第二章向我们展示了这是词法作用域的核心。

So, the best way to think about things is that all declarations, both variables and functions, are processed first, before any part of your code is executed.

所以，对于所有变量，不论是变量还是方法，最好把他们想成，他们会你的代码被执行之前的任何一步先被处理。

When you see `var a = 2;`, you probably think of that as one statement. But JavaScript actually thinks of it as two statements: `var a;` and `a = 2;`. The first statement, the declaration, is processed during the compilation phase. The second statement, the assignment, is left **in place** for the execution phase.

当你看见 `var a = 2;`,你可能把他认为是一个表达式。但是JavaScript实际上把他认为是两个表达式:`var a;`和`a = 2;`。第一个表达式，是一个声明，在编译阶段被处理。第二个表达式，是赋值表达式，在他的位置上在执行阶段执行。

Our first snippet then should be thought of as being handled like this:

我们的第一个片段我们可以把他想象成会像接下来这样处理:

```js
var a;
```
```js
a = 2;

console.log( a );
```

...where the first part is the compilation and the second part is the execution.

哪里第一部分是编译第二部分是执行。

Similarly, our second snippet is actually processed as:

类似的，我们的第二段代码实际上会这样处理：

```js
var a;
```
```js
console.log( a );

a = 2;
```

So, one way of thinking, sort of metaphorically, about this process, is that variable and function declarations are "moved" from where they appear in the flow of the code to the top of the code. This gives rise to the name "Hoisting".

所以，关于这个过程，可以把他想象成，类似比喻的方式，变量和方法声明会从在某些代码流中出现"被移动"到代码的顶部。这就有了名字"提升"。

In other words, **the egg (declaration) comes before the chicken (assignment)**.

换句话来说，**先有蛋(声明)然后有鸡(赋值)**。

**Note:** Only the declarations themselves are hoisted, while any assignments or other executable logic are left *in place*. If hoisting were to re-arrange the executable logic of our code, that could wreak havoc.

**注意：** 只有声明自己会被提升，任何赋值或者其他可执行的逻辑都会留在原地。如果提升会重新安排我们代码的可执行逻辑，那会造成大灾难。

```js
foo();

function foo() {
	console.log( a ); // undefined

	var a = 2;
}
```

The function `foo`'s declaration (which in this case *includes* the implied value of it as an actual function) is hoisted, such that the call on the first line is able to execute.

方法`foo`的声明(在这个例子中*包含*了一个隐含的值实际作为一个方法)被提升了，所以在第一行的调用时可以执行的。

It's also important to note that hoisting is **per-scope**. So while our previous snippets were simplified in that they only included global scope, the `foo(..)` function we are now examining itself exhibits that `var a` is hoisted to the top of `foo(..)` (not, obviously, to the top of the program). So the program can perhaps be more accurately interpreted like this:

注意到提升是**依照作用域**来的是很重要的。虽然我们的前一段代码被简化成只包含全局作用域，我们需要测试的方法`foo(..)`自己展示了，变量`var a`被提升到`foo(..)`的顶端(显然，不是程序的顶部)。所以程序也许可以更准确的像这样解释：

```js
function foo() {
	var a;

	console.log( a ); // undefined

	a = 2;
}

foo();
```

Function declarations are hoisted, as we just saw. But function expressions are not.

就像我们之前看到的，方法声明会被提升。但是方法表达式不会。

```js
foo(); // not ReferenceError, but TypeError!

var foo = function bar() {
	// ...
};
```

The variable identifier `foo` is hoisted and attached to the enclosing scope (global) of this program, so `foo()` doesn't fail as a `ReferenceError`. But `foo` has no value yet (as it would if it had been a true function declaration instead of expression). So, `foo()` is attempting to invoke the `undefined` value, which is a `TypeError` illegal operation.

变量标识符`foo`被提升了然后依附于程序中包围的作用域(全局)，所以`foo()`的结果不是失败然后抛出一个`ReferenceError(引用错误)`错误。但是`foo`还没有值(如果他是一个方法声明而不是一个方法表达式那么他就会有值)。所以，`foo()`企图触发一个`undefined`值，这是一个`TypeError(类型错误)`的不合法操作。

Also recall that even though it's a named function expression, the name identifier is not available in the enclosing scope:

即使方法表达式被取了名字，这个标识符的名字在包围的作用域中也不是变量：

```js
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
	// ...
};
```

This snippet is more accurately interpreted (with hoisting) as:

这个片段可以更精确的被解释(提升)为：

```js
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
	var bar = ...self...
	// ...
}
```

## Functions First

## 方法优先

Both function declarations and variable declarations are hoisted. But a subtle detail (that *can* show up in code with multiple "duplicate" declarations) is that functions are hoisted first, and then variables.

方法声明和变量声明都会提升。但是有个微妙的细节(*可以* 在代码中通过声明多个重复变量来显示)是方法会先被提升，然后是变量。

Consider:

考虑一下：

```js
foo(); // 1

var foo;

function foo() {
	console.log( 1 );
}

foo = function() {
	console.log( 2 );
};
```

`1` is printed instead of `2`! This snippet is interpreted by the *Engine* as:

`1` 会被输出而不是 `2`！这个片段被引擎解释为：

```js
function foo() {
	console.log( 1 );
}

foo(); // 1

foo = function() {
	console.log( 2 );
};
```

Notice that `var foo` was the duplicate (and thus ignored) declaration, even though it came before the `function foo()...` declaration, because function declarations are hoisted before normal variables.

注意`var foo`是一个重复(因此被忽略)声明，尽管他在`function foo()...`之后声明，因为方法声明会被提升到普通声明之后。

While multiple/duplicate `var` declarations are effectively ignored, subsequent function declarations *do* override previous ones.

虽然多个/重复 `var` 声明会被忽略，但是按照顺序方法声明覆盖了前面一个。

```js
foo(); // 3

function foo() {
	console.log( 1 );
}

var foo = function() {
	console.log( 2 );
};

function foo() {
	console.log( 3 );
}
```

While this all may sound like nothing more than interesting academic trivia, it highlights the fact that duplicate definitions in the same scope are a really bad idea and will often lead to confusing results.

虽然这一切似乎听起来只不过是有趣的学术上的琐事，但是要强调的事实是在同个作用域中重复定义真的是个不好的主意而且将会引起令人疑惑的结果。

Function declarations that appear inside of normal blocks typically hoist to the enclosing scope, rather than being conditional as this code implies:

方法声明在普通的块中出现通常会提升到包围他的作用域中，而不是像这段代码暗示的传统方式一样：

```js
foo(); // "b"

var a = true;
if (a) {
   function foo() { console.log( "a" ); }
}
else {
   function foo() { console.log( "b" ); }
}
```

However, it's important to note that this behavior is not reliable and is subject to change in future versions of JavaScript, so it's probably best to avoid declaring functions in blocks.

然而，注意到这个举止是不可信任的而且根据未来的JavaScript版本修改会受影响是很重要的，所以最好尽可能避免在块中声明方法。

## Review (TL;DR)

## 回顾(TL;DR)

We can be tempted to look at `var a = 2;` as one statement, but the JavaScript *Engine* does not see it that way. It sees `var a` and `a = 2` as two separate statements, the first one a compiler-phase task, and the second one an execution-phase task.

我们会被诱惑把`var a = 2;`当成一个表达式，但是JavaScript *引擎* 不会把他认为这个样子。他把`var a`和`a = 2`作为两个完全分开的表达式，第一个是编译阶段的任务，而第二个是执行阶段的任务。

What this leads to is that all declarations in a scope, regardless of where they appear, are processed *first* before the code itself is executed. You can visualize this as declarations (variables and functions) being "moved" to the top of their respective scopes, which we call "hoisting".

作用域中的所有声明会如何被处理，不管在哪里出现，会在代码自己执行之前*首先*被处理。你可以把这个视作变量(变量和方法)会*被移动*到各自的作用域的顶上，我们叫做"提升"。

Declarations themselves are hoisted, but assignments, even assignments of function expressions, are *not* hoisted.

变量他自己会被提升，但是赋值，即使是方法表达式的赋值，都*不会*被提升

Be careful about duplicate declarations, especially mixed between normal var declarations and function declarations -- peril awaits if you do!

小心重复的声明，特别是普通的var声明和方法声明混合的情况 -- 如果你这么做的话会面临严重的危险！

