# JavaScript优化方案
# (！！！这是之前的版本待完善)
## 1	概述
本文档对JavaScript前端编码重构的方案，重构的目的是让代码在保证功能和效率的同时更具有可维护性。

本文余下的部分会使用js作为JavaScript的简写形式。
主要内容包含js编码的基本原则，js子目录结构划分，js模块化处理，最后介绍了和php耦合的js代码的处理方式。

## 2	基本原则
1.	语言编码统一使用UTF-8 。
2.	变量函数名用camel方式命名，首字母小写。变量名尽量具体，避免不太通用的缩写。构造函数一般首字母大写。常量全部大写用下划线隔开单词（即不采用camel的命名方式）。
3.	合适的地方加上必要的注释。每个文件第一行加上一行注释说明该文件的用途，有必要时可说明该文件对其他文件的依赖；公有函数前面加上对该函数说明的注释，主要包括函数的用途，参数及返回值；复杂的函数顶部加上函数实现的逻辑说明。（参见图1）
```javascript
/**
 * 组件名称：示例
 * 组件详细说明：示例
 * 对应phtml：无
 * 其他说明：
 */

YUBA.example = (function ($, YUBA) {

    var config = {
        const:{
            TOP_WIDTH:50
        },
        dom:{
            $topicContainer:$("#topicContainer")
        }
    };

    function topicContainerHandler(){
        var $this=$(this);
        $this.width=config.const.TOP_WIDTH;
    }

    /**
     * 减法
     * @param {number} 减少
     * @param {number} 被减数
     * @returns {number}
     */
    function subtract(a,b){
        return a-b;
    }

    function init () {
        var $topicContainer=config.dom.$topicContainer;
        $topicContainer.on("click",topicContainerHandler);

        //事件代理，ajax加载的li
        $topicContainer.on("click","li",function(){

        });
    }

    return {
        init: init,
        subtract:subtract,
    };
})(jQuery, YUBA);

```

## 3	目录划分
在项目的js目录下添加5个字文件夹对不同类的文件进行管理。这样就不会让js目录下有太多的js文件，详细目录如图2。

![alt 图2][js-2]
+ dist是目标文件夹，主要是页面需要引用的js文件。主要包括jquery.js和每个页面页面对应的一个压缩后的js文件。这个压缩的js文件是其他4个文件夹的js文件合并生成的。
+ plugs是插件文件夹。主要包含一些jquery插件或者自己写或第三方一些功能js文件。编辑器的所有文件和用于首页搜索的jquery.searcher.js放在这个目录下。
+ common是公共文件夹。主要是一些页面的公共的js文件。比喻每个页面都需要的common.js和notification.js。
+ components是组件文件夹。把页面拆成的每个组件对应的js文件会放在这个文件夹中。比喻每个页面首位对应的js文件header.js 和footer.js；首页三个图片对应的js文件pictureGallery.js等。所有组件见图3。

![alt 图3][js-3]
pages文件夹，针对每个页面回有一个js文件，这些js文件是该页面除组件外的js代码。一般在这些js中调用各个组件的init方法。鱼吧目前所有的页面见图4。

![alt 图4][js-4]

## 4	js模块化
### 4.1	优点就前提
+ 让js和功能模块对应，使得js易于管理，修改的时候也知道这部分js代码主要影响的部分。减少不同的js代码片段相互的干扰。
+ 模块用自执行函数包装起来，减少对全局变量的污染。同时还可以隐藏私有变量和方法。
+ 在common.js里面定义一个全局变量“yuba”，然后把各个模块作为这个全局变量的属性，各个模块的共有属性和方法作为模块的属性和方法（在4.2模块构成部分会有更详细的说明）。
### 4.2	模块构成
通过自执行函数来包装一个模块，全局变量“YUBA”作为参数传入，在函数内部为“YUBA”添加模块，然后把该模块的共有方法和属性放在模块内部。
例如图1，在YUBA对象上创建了一个example模块，为这个example模块添加了一个共有的方法init。这样在其他位置就可以通过代码 YUBA.example.init() 调用这个方法。

```javascript
/**
 * 组件名称：示例
 * 组件详细说明：示例
 * 对应phtml：无
 * 其他说明：
 */

YUBA.example = (function ($, YUBA) {

    var config = {
        const:{
            TOP_WIDTH:50
        },
        dom:{
            $topicContainer:$("#topicContainer")
        }
    };

    function topicContainerHandler(){
        var $this=$(this);
        $this.width=config.const.TOP_WIDTH;
    }

    /**
     * 减法
     * @param {number} 减少
     * @param {number} 被减数
     * @returns {number}
     */
    function subtract(a,b){
        return a-b;
    }

    function init () {
        var $topicContainer=config.dom.$topicContainer;
        $topicContainer.on("click",topicContainerHandler);

        //事件代理，ajax加载的li
        $topicContainer.on("click","li",function(){

        });
    }

    return {
        init: init,
        subtract:subtract,
    };
})(jQuery, YUBA);

```
## 5	和php耦合的JavaScript处理

目前的代码中，部分php代码和JavaScript混杂在一起。可以通过两种方式来解耦。第一种把JavaScript代码段包装成某件的共有方法，然后在php中执行少许的JavaScript方法调用。第二种添加隐藏的html标签来存储php中用到的变量，通过js 取对应的变量执行相应的代码。本次js代码优化主要采用第一种方式，第二种方式作为补充。

ps:目录结构及文件说明参见下面[链接][1]

[1]: http://naotu.baidu.com/file/6503ff309d7419ae6914a0935f3c8461?token=a394f5ed2b8c4821



[js-2]: images/image002.png "图2" 
[js-3]: images/image003.png "图3" 
[js-4]: images/image004.png "图4" 
[js-5]: images/image005.png "图5" 