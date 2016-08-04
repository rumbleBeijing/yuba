### 原文地址：http://davidshariff.com/blog/javascript-scope-chain-and-closures/#more-271

从我的[上篇文章](http://blog.csdn.net/tangxiaolang101/article/details/52087239)中，我们知道每个函数都会有一个关联的`execution context`，`execution context`包含一个变量对象`variable object[VO]`。变量对象由函数的局部变量、内部函数和参数声明组成。  
每个`execution context`的作用域链（`scope chain`）属性简单来说就是当前context的[VO]和所有父级的词法作用域的[VO]。  
Scope = VO + All Parent VOs  
Eg: scopeChain = [[VO] + [VO1] + [VO2] + [VO n+1]];  
#### 定义作用域链的变量对象
#### [VO]s
现在我们知道作用域链的第一个[VO]属于当前的`execution context`，通过查找父级context的作用域链，我们能够找到其余的父级[VO]：

```
function one() {

    two();

    function two() {

        three();

        function three() {
            alert('I am at function three');
        }

    }

}

one();
```
![image](http://davidshariff.com/blog/wp-content/uploads/2012/06/stack14.jpg)

这个例子直截了当的表明了这种情况。从`global context` 开始，我们调用了 `one()`, `one()`调用了`two()`, 然后又调用了`three()`。在函数 `tree()`内，执行了提示输出。上面的图片展示出了函数 `three()` 在执行 `alert('I am at function three')` 时的调用栈。此时，我们可以将 `scope chain` 理解为下面这样：  
`three() Scope Chain = [[three() VO] + [two() VO] + [one() VO] + [Global VO]]`
#### 词法作用域
JavaScript的一个重要特征就是解释器使用了词法作用域，而不是动态作用域。复杂点来说，就是所有内部函数都会根据代码位置，静态地绑定到父级`context`上。 

在我们上面的例子中，内部函数如何被调用并没有关系， `three()`总是会被静态地绑定到 `two()`上，`two`也会静态地绑定到`one`，等等。这样导致的连锁反应就是通过静态绑定作用域链，内部函数可以访问到外部函数的 `VO`。  
很多开发者困惑的源头就是词法作用域。我们知道函数每次调用将会创建新的 `execution context`并且关联 [VO]，`VO`确定了变量在当前context中的取值。  
用动态作用域或者是运行时变量绑定的思想去思考浏览器词法作用域的行为，会导致意想不到的结果。来看下面的例子：

```
var myAlerts = [];

for (var i = 0; i < 5; i++) {
    myAlerts.push(
        function inner() {
            alert(i);
        }
    );
}

myAlerts[0](); // 5
myAlerts[1](); // 5
myAlerts[2](); // 5
myAlerts[3](); // 5
myAlerts[4](); // 5
```

匆匆一瞥，JavaScript的代码将会在i每次增加1的时候调用alert(i),然后分别输出1, 2, 3, 4, 5。  
这是大家普遍会感到困惑的地方。函数 `inner` 在 `global context`中创建，因此它的作用域链被静态地绑定到 `global context`。
代码的11行到15行调用 `inner()`,它将会在 `inner.ScopeChain`中查找i，为i确定值，然后将i定位在 `global context`。在这种情况下，每次调用的时候， `i` 已经增加到5了，所以 `inner()`每次调用输出的结果都是5。这种通过 `VOs` 让每个context包含活跃变量的静态绑定，经常让开发者大为惊骇。  

#### 变量值的分析
下面这个例子弹出变量a, b, c的和，结果是6。

```
function one() {

    var a = 1;
    two();

    function two() {

        var b = 2;
        three();

        function three() {

            var c = 3;
            alert(a + b + c); // 6

        }

    }

}

one()​;​
```
第14行很有钱，乍一看 a 和 b 都不在函数 three 内，代码如何继续执行呢？ 为了理解解释器如何执行这行代码，我们需要看看第14行代码执行时，函数 three 的作用域链是什么样的：
![image](http://davidshariff.com/blog/wp-content/uploads/2012/06/scopechain1.png)
当解释器执行第14行代码 `alert(a + b + c)`, 首先确定 a 的值，通过在作用域链中查找并且检查第一个变量对象——`three's [VO]`。如果a存在于`three's [VO]`，但是找不到具有 a 这个名称的属性，就会向下移动，检查下一个 `[VO]`。  
解释器将会在 [VO]序列中查找变量的名称，如果找到了，就会将它返回给执行代码，如果没有，程序将会抛出 `ReferenceError`的错误。因此，上面的例子中，变量a, b, c的值都能够在函数 three 的作用域链中确定出来。
#### 闭包是如何执行的
在JavaScript中，闭包经常被看作是某种神奇的独角兽，只有高级开发者才能真正的理解它。但是，事实上，闭包仅仅是作用域链的简单理解。一个闭包，就像 *Crockford* 说的一样，很简单：  
*内部函数经常可以访问到外部函数的变量和参数，即使外部函数已经即使运行了…*  
下面是一个闭包的例子：

```
function foo() {
    var a = 'private variable';
    return function bar() {
        alert(a);
    }
}

var callAlert = foo();

callAlert(); // private variable
```
在 `global context`中有一个名字为 `foo()` 的函数，和一个变量名为 'callAlert'的变量，它的值是函数 `foo()`的返回值。经常让开发者感到吃惊和困惑的是私有变量 a ，在函数'foo()'执行完成后仍然可以被访问到。  
不管怎么样，如果我们着眼于每个context的细节，我们将会看到如下展示：

```
// Global Context when evaluated
global.VO = {
    foo: pointer to foo(),
    callAlert: returned value of global.VO.foo
    scopeChain: [global.VO]
}

// Foo Context when evaluated
foo.VO = {
    bar: pointer to bar(),
    a: 'private variable',
    scopeChain: [foo.VO, global.VO]
}

// Bar Context when evaluated
bar.VO = {
    scopeChain: [bar.VO, foo.VO, global.VO]
}
```
现在，我们可以看到，通过调用 `callAlert()` 我们得到函数 `foo()` ，它又返回了指向 `bar()`的指针。当进入 `bar()` 时， bar 变量对象的作用域链是这样的：`[bar.VO, foo.VO, global.VO]`。  
解释器首先在 `bar.VO.scopeChain` 的第一个 VO 中查找名为 `a` 的属性，但是没有找到，所以，继续移动到下一个 VO， `foo.VO`。  
当找到这个属性后，就把值返回给 bar 的context，这也就解释了为什么在函数 `foo()`已经完成执行后， `alert()` 给我们的值仍然是 `private variable`。  
这篇文章到这里呢，我们已经讨论了作用域链和词法环境的很多细节，了解了闭包和变量赋值的工作原理。文章剩下的部分将会讨论一些相关的解决方案。  
#### 等等，原型链是如何影响变量赋值的？
JavaScript是原型语言，它的绝大部分元素除了 `null` 和 `undefined` ，基本都是对象（`objects`）。当试图访问一个对象的属性时，解释器会试图查找对象的属性来确定它的值。如果没有找到这个属性，解释器就会继续向上查找原型链，即对象的继承链，直到找到该属性或者是查找到原型链的尽头。  
这导致了一个有趣的问题，解释器确定对象属性值得时候是先用作用域链还是先用原型链呢？事实上是两个都用。当确定一个属性的值的时候，作用域链先用来定位 object 的位置。当 object 被找到后，再通过 object 的原型链查找属性名。举个例子：

```
var bar = {};

function foo() {

    bar.a = 'Set from foo()';

    return function inner() {
        alert(bar.a);
    }

}

foo()(); // 'Set from foo()'
```
代码第五行全局对象 bar 的属性 a 被赋值为 `Set from foo()`。解释器进入作用域链中进行查找，不出所料，在全局上下文中找到了 `bar.a`。现在让我们想一下下面的代码：

```
var bar = {};

function foo() {

    Object.prototype.a = 'Set from prototype';

    return function inner() {
        alert(bar.a);
    }

}

foo()(); // 'Set from prototype()'
```
在运行时，我们调用函数 `inner()`，它会用到 `bar.a` ，所以解释器会在作用域链中查找对象 bar 。当然，解释器在全局上下文中找到了 bar。然后它会继续在 bar 上查询属性 a。但是，bar 上从来就没有设置过 a，所以，解释器会继续查询原型链，然后再 `Object.prototype` 上找到了属性 a。  
这就是标签解析的正确行为——在作用域中定位 `object` ，在对象的原型链中查找属性，直到找到该属性，或者没有找到，就会返回 `undefined` 了。  
#### 什么时候使用闭包
对于JavaScript来说，闭包是一个很强大的概念，许多常用的解决方案都用到了它：  
##### 封装
可以对外部隐藏实现细节，外露一个公共操作接口。适用于 `module pattern` 或者是 `revealing module pattern`。
Module Export

```
var MODULE = (function () {
	var my = {},
		privateVariable = 1;

	function privateMethod() {
		// ...
	}

	my.moduleProperty = 1;
	my.moduleMethod = function () {
		// ...
	};

	return my;
}());
```
Revealing Module Pattern

```
var MODULE = (function () {
  // 私有变量及函数
  var x = 1;
  function f1() {}
  function f2() {}

  return {
    public_method1: f1,
    public_method2: f2
  };
}());
```
##### 回调函数
也许对于闭包的使用，最强大的还是回调函数。在浏览器中，JavaScript是在一个单线程事件轮询中运行的，直到一个事件完成，其他事件才会开始执行。回调函数允许我们延迟调用一个函数，用一种非阻塞的方式响应事件完成状态。举个例子，当向服务器做 AJAX 请求时，使用回调函数处理请求的响应。
##### 闭包作为参数
我们也可以把闭包作为参数传入函数。这是一种很强大的函数范式，可以为复杂代码创建出更好的解决方案。例如排序函数，通过传入闭包作为参数，一个函数体，实现不同类型的数据排序。
#### 什么时候使用闭包
尽管闭包很强大，但是由于它存在一些性能隐患，所以要保守地去使用。
##### 作用连很长
多重嵌套函数是一个经典的使用标志，这种情况使用闭包可以解决一些性能问题。我们记得之前说过，每次需要确定一个变量值的时候，作用域链都需要被查询去查找变量的声明，变量在作用链的位置越深，需要查找的时间就越多。  
#### 垃圾回收
JavaScript 是一种 `grabage collected` 语言。这意味着开发人员可以不用像使用底层语言那样，去考虑内存管理的问题。自动垃圾回收机制经常会导致开发人员的应用出现性能不佳和内存泄漏的问题。  
不同的 JavaScript 引擎实现的垃圾回收也有细微的差别。因为 `ECMAScript` 并没有定义统一的实现标准。但是，所有引擎的实现理念都是为了追求更好的性能，泄漏更少的代码。通常来说，垃圾回收器会尝试释放掉不再被引用或者是已经无法被访问到的对象的内存。
#### 循环引用
循环引用这个词描述的是一个对象引用另一个对象，然后这个对象有重新指回了第一个对象。闭包尤其容易导致内存泄漏，因为内部函数能够引用定义在当前作用域上一层（依旧在作用域链中）中定义的变量，即使这个父级函数已经执行完成了。绝大部分浏览器引擎都能很好的处理这种情况（除了IE），你做开发的时候也没有什么值得去考虑的。  
老版本的IE，引用DOM元素的时候经常造成内存泄漏。为什么？ 在IE中， JavaScript 引擎和DOM都有自己独立的垃圾回收器。因此，当在JavaScript中引用DOM元素时，本地回收器切换的DOM，DOM的回收器又指回本地的，这样就造成了循环引用。  
#### 总结
从这一些年一起工作的开发人员来看，我发现大家对作用域链和闭包的概念都了解，但是并没有真正的深入探讨细节。我希望这篇文章能够帮助你了解这些基础概念，能够理解更多的细节和深层次的探讨。  
继续前行，用这些知识武装自己，在任何的情况下，都能理解你所编写的JavaScript代码时如何解析变量的。快乐的码。。。






