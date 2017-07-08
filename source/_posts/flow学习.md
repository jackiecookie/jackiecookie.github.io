---
layout: '[layout]'
title: flow学习
date: 2017-07-06 16:19:04
tags:
---

类似JavaScript这种动态语言，任何变量可以随时更改为任何类型的值。他不像强类型的语言，在变量声明之初就规定了变量的类型，任何情况下以任何方式尝试修改变量的类型都是不允许的。他的优点是灵活，但是另外一方面无疑增加了后期维护的成本，我们可以很容易在一些库的方法中发现大量用于类型检查的代码，类似下面的伪代码：

```
fn.init=function(selector){
  if ( typeof selector === "string" ) {
    ...
  }else if(typeof selector === "function"){
    ...
  }else{
    //参数可能是我们不支持的类型
    return ...;
  }
}
```
如果是上面的代码，我们可能很难很简单告诉其他开发人员这个方法支持的参数类型，可能需要借助注释的方式。但是这不直接而且没有约束力，所以很容易会在一个大项目中造成后期维护上的困难和难以察觉的bug。

而[folw](https://flow.org/en/)就是这样一个由facebook主导开发的用于检查JavaScript类型的工具，他的目标就是快速精准的找到可能因为类型引起的bug，他可以有效的管理变量的类型，也可以定义大对象，复杂对象，子对象，管理对象的属性类型。从而约束其他开发人员或者自己无意间传入参数的类型，避免多人协作或者自己开发时发生后期因为类型而发生难以察觉的bug。

先用官方的小例子来简单的展示一下flow的代码是什么样子的。
```
// @flow
function square(n: number): number {
  return n * n;
}

square("2"); // Error!
```
他规定了square方法的参数n只能是number类型，且返回值也只能是number类型。任何不是number类型的参数或者返回值都会在flow检查时抛出一个错误。

你可以通过npm或者Yarn来安装folw:
``` bash
npm install --save-dev flow-bin
```
除了安装flow外还需要安装一个flow的编译器,这个部分你也可以选择Balel或者是flow-remove-types
``` bash
npm install --save-dev babel-cli babel-preset-flow
```
或者
``` bash
npm install --save-dev flow-remove-types
```
下面是flow的几个主要命令
``` bash
flow init  //初始化flow
flow check  //检查所有需要检查的文件打印出结果
flow status //开启flow监控任务,监控文件修改进行检查
```

写flow代码首先要告诉Flow这个js文件为flow代码让他对其进行类型检查，在js文件的开始部分使用注释的方式//@flow ，否则这个文件将不会被检查。除非你在.flowconfig的[options]区块中设置all为true。

看下面的flow代码
```
function method(x: Number, y: String, z: Boolean) {
    // ...
}

method(new Number(42), new String("world"), new Boolean(false)); //成功
method(42, "world", false); //失败
```
**注意:** 这里的Boolean和boolean,String和string,Number和number之间都是不同的对象，关于上面的代码Flow在进行检查后会抛出下面的错误：

```
index.js:10
 10: method(42, "world", false);
            ^^ number. This type is incompatible with the expected param type of
  5: function method(x: Number, y: String, z: Boolean) {
                        ^^^^^^ Number

index.js:10
 10: method(42, "world", false);
                ^^^^^^^ string. This type is incompatible with the expected param type of
  5: function method(x: Number, y: String, z: Boolean) {
                                   ^^^^^^ String

index.js:10
 10: method(42, "world", false);
                         ^^^^^ boolean. This type is incompatible with the expected param type of
  5: function method(x: Number, y: String, z: Boolean) {
                                              ^^^^^^^ Boolean


Found 3 errors
```
flow支持检查的类型很多，可以是原始类型：
- number
- string
- boolean
- 等等
也可以约束直接的字面量，类似：
```
// @flow
function acceptsTwo(value: 2) {
  // ...
}
acceptsTwo(2);   // 成功!
acceptsTwo(3);   // 错误!
```
或者是这样
```
// @flow
function getColor(name: "success" | "warning" | "danger") {
  switch (name) {
    case "success" : return "green";
    case "warning" : return "yellow";
    case "danger"  : return "red";
  }
}

getColor("success"); // 成功!
getColor("danger");  // 成功!
getColor("error");   // 错误!
```
还可以是像高级语言里面的泛型：
```
function identity<T>(value: T): T {
    return value;
}
```
很酷吧，当你想接受任何类型的的参数时你可以使用mixed，像这样`functionstringify(value: mixed) {  // ...} `。但是需要强调的是参数为mixed的方法有一个限制，**方法的返回值必须是经过类型检查的，随意返回值将会视为是一个错误。像这样：**
```
// @flow
function stringify(value: mixed) {
  return "" + value; // 错误，不被允许!
}

function stringify(value: mixed) {
  if (typeof value === 'string') {
    return "" + value; // 因为进行了类型检查，flow就知道value只能是string类型!
  } else {
    return "";
  }
}
```
如果你确实不想限制类型也不想flow干预你返回的值那么你可以使用`any`关键字，但是他是非安全的，应该尽可能的不要使用他。

你也可以根据你的需要约束变量或者对象属性的类型：
```
// @flow
var obj: {
  foo: ?number
}={
  foo: 1   //成功
}

obj= {
  foo: undefined  //成功
}

obj= {
  foo: null    //成功
}

var foo: number= obj.foo  //错误
```
?number表示foo的值可以是number,null或者是undefined,但是foo变量的值必须是一个number,flow会替你检查obj.foo类型的所有可能性，所以flow将会在这里抛出一个错误。
