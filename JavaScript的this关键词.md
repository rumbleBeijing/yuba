#### 译文
原文链接：http://davidshariff.com/blog/javascript-this-keyword/#first-article  
`this` 关键词是 JavaScript 一个常用的特性，但是它同时也是这门语言中最容易让人困惑或者误解的一个点。this 实际上是什么意思？如何去确定呢？  
这篇文章尽力用一种清晰的方式来解决大家的困惑。  
'this' 关键字对于其它程序化编程语言并不新奇，它经常指向一个类的构造函创建的新对象。举个例子：如果我有一个 `Boat()` 类，它含有一个 `moveBoat()` 方法，当在 moveBoat() 方法内部引用 `this` 的时候，我们实际上访问的是新创建的对象 Boat().  
在JavaScript中，当使用new关键字的时候，我们也有构造函数这么个概念。但是，这并不是唯一的规则， this 经常会根据不同的execution context 指向不同的对象。如果你对execution context不是很清楚的话，我推荐你阅读我的[另一篇文章](http://blog.csdn.net/tangxiaolang101/article/details/52087239)。说的够多了，让我们来看一些JavaScript的例子：  

```
// global scope

foo = 'abc';
alert(foo); // abc

this.foo = 'def';
alert(foo); // def
```
当我们在全局上下文中（不再函数内部）使用 this 关键字时，this 总是指向全局对象。现在让我们看看 this 在函数内部的情况：  

```
var boat = {
    size: 'normal',
    boatInfo: function() {
        alert(this === boat);
        alert(this.size);
    }
};

boat.boatInfo(); // true, 'normal'

var bigBoat = {
    size: 'big'
};

bigBoat.boatInfo = boat.boatInfo;
bigBoat.boatInfo(); // false, 'big'
```
上面这个例子中 this 如何确定？ 我们可以看到，boat 对象有一个属性size，和一个方法 boatInfo()。在 boatInfo()内，判断 this 的值是不是 boat 对象，并且用alert输出属性this的值。因此，我们使用 boat对象调用函数的时候可以看到，this 就是 boat对象，boat的size属性值是normal。  
然后我们创建了另一个对象 bigBoat，bigBoat也有一个属性size，值是big。但是，bigBoat对象没有boatInfo()方法，所以我们使用 `bigBoat.boatInfo = boat.boatInfo`从boat那里复制一个。现在，我们调用 bigBoat.boatInfo()，我们可以看到 this 的值不再是 boat并且属性size的值现在是big了。为什么会这样？boatInfo()内部如何改变了this的值？  
首先，你必须认识到一点，在任何函数内部，this的值永远不是静态的。它会在你每次调用函数并且真正执行函数代码之前被确定。this的值是由函数调用时的父级作用域提供的，更重要的是，如何写实际函数的语法。  
当一个函数被调用的时候，我们立即看 `()`左边的部分。如果在括号左边，我们能看到一个引用，那么 this 的值就是调用这个函数的对象，否则就是全局对象。让我们看一些例子：  

```
function bar() {
    alert(this);
}
bar(); // global - because the method bar() belongs to the global object when invoked

var foo = {
    baz: function() {
        alert(this);
    }
}
foo.baz(); // foo - because the method baz() belongs to the object foo when invoked
```
现在为止，一切都还讲的通，都很清晰。我们进一步复杂化，通过两种不同的调用方式，改变 this 在同一函数中的值： 

```
var foo = {
    baz: function() {
        alert(this);
    }
}
foo.baz(); // foo - because baz belongs to the foo object when invoked

var anotherBaz = foo.baz;
anotherBaz(); // global - because the method anotherBaz() belongs to the global object when invoked, NOT foo
```
这里，我们可以看到，用两种不同的方式对函数 baz() 进行同步调用时，this 的值是不同的。现在，让我们看看深层嵌套对象中 this 的值： 

```
var anum = 0;

var foo = {
    anum: 10,
    baz: {
        anum: 20,
        bar: function() {
            console.log(this.anum);
        }
    }
}
foo.baz.bar(); // 20 - because left side of () is bar, which belongs to baz object when invoked

var hello = foo.baz.bar;
hello(); // 0 - because left side of () is hello, which belongs to global object when invoked
```
另一个经常被问到的问题是 this 关键字在事件句柄中是如何被确定的？答案是：this 在事件句柄中总是指向被触发的元素。让我们看看下面这个例子：

```
<div id="test">I am an element with id #test</div>function doAlert() {
    alert(this.innerHTML);
}

doAlert(); // undefined

var myElem = document.getElementById('test');
myElem.onclick = doAlert;

alert(myElem.onclick === doAlert); // true
myElem.onclick(); // I am an element
```
这里我们可以看到，当doAlert()第一次被调用的时候，提示undefined，因为 doAlert() 属于全局对象。然后我们写了 `myElem.onclick = doAlert`，把 doAlert() 函数拷贝赋值给了myElem 的onclick()事件。这意味着当onclick被激活的时候，它是myElem的一个方法，准确的来说，这意味着 `this` 的值将会是myElem对象。  
关于这个话题，我想说的最后一个点是，this的值也可以通过call()和apply()手动设置，推翻上面我们所讨论的一切。同样有趣的是，当在构造函数内部调用 this 的时候，this 指向的是新创建的对象实例。原因是，构造函数被 new 关键字调用的时候，会创建一个新的对象，构造函数中的this总是指向那个刚刚被创建的对象。

### 总结
希望今天的这篇文章能够解决大家关于this关键字的疑惑，取得进步，并且能够总是知道this正确的取值。我们现在都了解了，this永远都不会是静态的，它的值依赖于它的函数的调用方法。
