---
title: 深入探讨JavaScript的执行环境和栈（What is the Execution Context & Stack in JavaScript）
date: 2016-08-01 17:05:38
tags: [JavaScript, 翻译]
---


原文网址：http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/

这篇文章我将会深入地讨论JavaScript中最根本的一部分——Execution Context(执行上下文)。在文章结束的时候，你应该对解释器的工作原理有一个比较清晰的理解，对于为什么会存在‘变量提升’，它们的值又是如何被真正确定的这些问题有一个正确的答案。

#### 什么是Executin Context(执行上下文)
 当JavaScript代码执行的时候，执行环境是很重要的，它可能是下面三种情况中的一种：
* 全局 code（Global code）——代码第一次执行的默认环境
* 函数 code（Function code）——执行流进入函数体
* Eval code（Eval code）——代码在eval函数内部执行

在网上你能够读到许多关于作用域的资料，这篇文章的目的是让事情变得简单些。让我们来思考下execution context这个词，它与当前代码的环境 / 作用域是等价的。好了，说的够多了，让我们来看一个包含global和function / local context的例子吧。

![image](http://davidshariff.com/blog/wp-content/uploads/2012/06/img1.jpg)  
 这里没有什么特别的地方，我们有一个global context被紫色的框框着，还有三个不同的function context，分别被绿色、蓝色、橘色的框框着。在你的程序中，有且仅能有一个global context，并且它能够被任何其他的context访问到。  
你能够拥有任意多个function context，并且每个函数被调用的时候都会生成一个新的context。它会生成一个私有作用域，并且在它内部声明的任何东西都不能直接在它的外部访问。就像上面的例子，一个函数可以直接访问它外面的变量，但是外部的context就不能直接访问在内部声明的变量或者函数。为什么会这样？如何准确的理解这段代码的执行？  
** 以上是文章摘要 阅读更多请点击------>右下角的more ** <!--more--> 以下是余下全文 
#### Execution Context Stack(执行上下文栈)
 浏览器中的JavaScript解释器是单线程的。意思就是说在浏览器中，同一时间只能做一件事，其它的行为和事件都会在Execution Stack中排队。下面这个图表就是一个单线程栈的抽象描述：  
![image](http://davidshariff.com/blog/wp-content/uploads/2012/06/ecstack.jpg)

 我们已经知道，当浏览器第一次加载你的script的时候，默认进入global execution context。如果，你在全局代码中调用了函数，程序序列流就会进入被调用的函数中，生成一个新的execution context并且把它压入execution stack的顶部。  
 如果你在当前函数内调用其他函数，会发生同样的事情。代码的执行流会进入内部函数，生成一个新的execution context,并且将它压入existing stack。浏览器总是会执行stack顶部的executin context，当被执行的函数上下文执行完成后，它将会弹出栈顶，然后将控制权返回给当前栈中的下一个对象。下面是一个循环函数执行栈的例子：

```
(function foo(i) {
    if (i === 3) {
        return;
    }
    else {
        foo(++i);
    }
}(0));
```
![image](http://davidshariff.com/blog/wp-content/uploads/2012/06/es1.gif)

函数`foo`递归调用了3次，每次 i 增长1。函数 `foo` 每次调用后，都出生成一个新的execution context。当一个context执行完成后，它就会出栈并且把控制权返回给下面的context，直到再次到达`global context`。
##### 关于`execution stack`有5个关键点需要记住：
* 单线程
* 同步执行
* 1个`Global context`
* 无限制的函数context
* 每个函数调用都会创建新的`execution context`，即使是自己调用自己。
#### Execution Context的细节
现在我们知道了函数每次被调用的时候，一个新的`execution context`就会被创建。无论如何，在JavaScript解释器内部，每次调用执行`execution context`都有两个阶段：
1. **创建阶段**【在函数被调用的时候，但是内部代码执行之前】
   * 创建Scope Chain
   * 创建变量、函数和参数
   * 确定 `"this"` 的值
2. 激活 / 代码执行阶段
 * 变量赋值、引用函数和解释 / 执行代码。  
每个`execution context`在概念上可以用一个对象来表示，这个对象有三个属性：

```
executionContextObj = {
    'scopeChain': { /* variableObject + all parent execution context's variableObject */ },
    'variableObject': { /* function arguments / parameters, inner variable and function declarations */ },
    'this': {}
}
```
#### 执行对象 / 变量对象【AO/VO】
这个`executionContextObj`在函数被调用的时候创建，但是是在真实函数代码被执行之前。这个就可以理解为第一阶段，创建阶段（`Creation Stage`）。在这里，解释器通过搜索函数的形参和传入的实参、本地函数的声明和本地变量的声明来创建`executionContextObj`。搜索的结果就是`executionContextObj`对象中的`variableOject`属性。  

**这里是解释器执行代码的一个伪综述：**
1. 找到调用函数的代码。
2. 在执行函数代码之前，创建`execution context`。
3. 进入创建阶段：
  * 初始化`Scope Chain`
  * 创建`variable object`
    * 创建实参对象（`arguments object`），检查context的形参（`parameters`），初始化参数的名称和参数值并且创建一份引用的拷贝。
    * 扫描context中的函数声明：
      * 为每一个函数在`varible object`上创建一个属性，属性名就是函数名，含有一个指向内存中函数的引用指针。
      * 如果函数名已经存在了，这个引用指针的值将会被重写。
    * 扫描context中的变量申明：
      * 为每一个变量在`variable object`上创建一个属性， 属性名就是变量名并且将变量的值初始化为undefined。
      * 如果变量名在`variable object`中已经存在，那就什么都不会发生，并且继续扫描。
* 确定context中 `"this"`的值。
4. 激活 / 代码执行阶段：
   * 运行 / 解释context中的函数代码，并且根据代码一行一行的执行，为变量赋值。

让我们来看一个例子：

```
function foo(i) {
    var a = 'hello';
    var b = function privateB() {

    };
    function c() {

    }
}

foo(22);
```
当调用`foo（22）`时，创建阶段（`creation stage`）时，`context`是下面这个样子：

```
fooExecutionContext = {
    scopeChain: { ... },
    variableObject: {
        arguments: {
            0: 22,
            length: 1
        },
        i: 22,
        c: pointer to function c()
        a: undefined,
        b: undefined
    },
    this: { ... }
}
```
因此，你可以看到，在创建阶段（`creation stage`）只负责对属性名称（变量名）的定义，但是并没有给它们赋值，当然这里有一个例外就是formal arguments / parameters(实参 / 形参)。当创建阶段完成以后，执行流进入函数内部,激活执行阶段（`execution stage`），然后代码完成执行，`context`是下面这个样子：

```
fooExecutionContext = {
    scopeChain: { ... },
    variableObject: {
        arguments: {
            0: 22,
            length: 1
        },
        i: 22,
        c: pointer to function c()
        a: 'hello',
        b: pointer to function privateB()
    },
    this: { ... }
}
```
#### 关于Hoisting（变量提升）
在网上你可以找到很多定义JavaScript中hoisting这个词的文献，解释变量和函数的声明在它们的作用域中被提前。但是，没有从细节上解释为什么会发什么这种现象。通过了解解释器如何创建`activation object`，就很容易知道这种现象发生的原因了。看下面这个例子：

```
(function() {

    console.log(typeof foo); // function pointer
    console.log(typeof bar); // undefined

    var foo = 'hello',
        bar = function() {
            return 'world';
        };

    function foo() {
        return 'hello';
    }

}());​
```
现在我们可以回答下面这些问题了：
* 在`foo`声明之前，为什么我们可以访问它？
  * 如果我们来跟踪`creation stage`, 我们知道在代码执行阶段之前，变量已经被创建了。因此在函数流开始执行之前，`foo`已经在`activation object`中被定义了。
* `foo` 被声明了两次，为什么 `foo` 最后显示出来是一个function，并不是undefined或者是string？
  * 尽管 `foo` 被声明了两次，我们知道，在创建阶段，函数的创建是在变量之前的，并且如果属性名在`activation object`中已经存在的话，我们是会简单的跳过这个声明的。
  * 因此，对 `function foo()`的引用在`activation object`上先被创建了，当解释器到达 `var foo` 时，我们会发现属性名 `foo` 已经存在了，因此代码什么都不会做，继续向下执行。
* 为什么 bar 是undefined？
  * bar实际上是一个变量，并且被赋值了一个函数的引用。我们知道变量是在创建阶段被创建的，但是它们会被初始化为undefined，所以bar是undefined。
#### 总结
希望现在你对JavaScript解释器如何执行你的代码能有一个好的理解了。理解execution context and stack会让你知道为什么你的代码有时候会输出和你最初期望不一样的值。





