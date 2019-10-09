# 前端面试题集

## TOC

## HTML

## CSS



### 描述下 CSS 盒模型及box-sizing作用



* 块级元素的大小由`width`、`height`、`padding`、`border`和`margin`决定。
* 默认情况下，`padding`和`border`不是元素`width`和`height`的组成部分
* 元素默认应用了`box-sizing: content-box`，元素的宽高只会决定内容（content）的大小。
* `box-sizing: border-box`改变计算元素`width`和`height`的方式，`border`和`padding`的大小也将计算在内。
* 元素的`height` = 内容（content）的高度 + 垂直方向的`padding` + 垂直方向`border`的宽度
* 元素的`width` = 内容（content）的宽度 + 水平方向的`padding` + 水平方向`border`的宽度

参考 [https://zhuanlan.zhihu.com/p/63962882](https://zhuanlan.zhihu.com/p/63962882) 



### CSS样式优先级?



浏览器通过优先级规则，判断元素展示哪些样式。优先级通过 4 个维度指标确定，我们假定以`a、b、c、d`命名，分别代表以下含义：

1. `a`表示是否使用内联样式（inline style）。如果使用，`a`为 1，否则为 0。
2. `b`表示 ID 选择器的数量。
3. `c`表示类选择器、属性选择器和伪类选择器数量之和。
4. `d`表示标签（类型）选择器和伪元素选择器之和。

优先级的结果并非通过以上四个值生成一个得分，而是每个值分开比较。`a、b、c、d`权重从左到右，依次减小。判断优先级时，从左到右，一一比较，直到比较出最大值，即可停止。所以，如果`b`的值不同，那么`c`和`d`不管多大，都不会对结果产生影响。比如`0，1，0，0`的优先级高于`0，0，10，10`。

当出现优先级相等的情况时，最晚出现的样式规则会被采纳。如果你在样式表里写了相同的规则（无论是在该文件内部还是其它样式文件中），那么最后出现的（在文件底部的）样式优先级更高，因此会被采纳。

在写样式时，我会使用较低的优先级，这样这些样式可以轻易地覆盖掉。尤其对写 UI 组件的时候更为重要，这样使用者就不需要通过非常复杂的优先级规则或使用`!important`的方式，去覆盖组件的样式了。



参考[https://zhuanlan.zhihu.com/p/63962882](https://zhuanlan.zhihu.com/p/63962882)



### display 都有哪些值及其用法？



* inline:
* block:
* inline-block:
* none:
* flex:



### position 都有哪些值？



- `static`：默认定位属性值。该关键字指定元素使用正常的布局行为，即元素在文档常规流中当前的布局位置。此时 top, right, bottom, left 和 z-index 属性无效。
- `relative`：该关键字下，元素先放置在未添加定位时的位置，再在不改变页面布局的前提下调整元素位置（因此会在此元素未添加定位时所在位置留下空白）。
- `absolute`：不为元素预留空间，通过指定元素相对于最近的非 static 定位祖先元素的偏移，来确定元素位置。绝对定位的元素可以设置外边距（margins），且不会与其他边距合并。
- `fixed`：不为元素预留空间，而是通过指定元素相对于屏幕视口（viewport）的位置来指定元素位置。元素的位置在屏幕滚动时不会改变。打印时，元素会出现在的每页的固定位置。fixed 属性会创建新的层叠上下文。当元素祖先的 transform 属性非 none 时，容器由视口改为该祖先。
- `sticky`：盒位置根据正常流计算(这称为正常流动中的位置)，然后相对于该元素在流中的 flow root（BFC）和 containing block（最近的块级祖先元素）定位。在所有情况下（即便被定位元素为 `table` 时），该元素定位均不对后续元素造成影响。当元素 B 被粘性定位时，后续元素的位置仍按照 B 未定位时的位置来确定。`position: sticky` 对 `table` 元素的效果与 `position: relative` 相同。

参考 [https://zhuanlan.zhihu.com/p/63962882](https://zhuanlan.zhihu.com/p/63962882) 



## JavaScript



### JavaScript 的类型有哪些？

* undefined

* null

* boolean

* number

* string

* object

* symbol
* bigint

前面都是旧的类型，最后 `symbol` 和 `bigint` 是新加的数据类型。

`symbol` 有什么特性？

```javascript
var a = Symbol('123');
var b = Symbol('123');
console.log(a === b);	// 结果？
```



参考

*  [JavaScript原子类型](https://developer.mozilla.org/en-US/docs/Glossary/Primitive) 
* [JavaScript数据类型](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#Data_types) 



### 使用let、var和const创建变量有什么区别



* 用var声明的变量的作用域是它当前的执行上下文，它可以是嵌套的函数，也可以是声明在任何函数外的变量。let和const是块级作用域，意味着它们只能在最近的一组花括号（function、if-else 代码块或 for 循环中）中访问
* var会使变量提升，这意味着变量可以在声明之前使用。let和const不会使变量提升，提前使用会报错
* 用var重复声明不会报错，但let和const会
* let和const的区别在于：let允许多次赋值，而const只允许一次

参考 [使用letvar和const创建变量有什么区别]([https://github.com/yangshun/front-end-interview-handbook/blob/master/Translations/Chinese/questions/javascript-questions.md#%E4%BD%BF%E7%94%A8letvar%E5%92%8Cconst%E5%88%9B%E5%BB%BA%E5%8F%98%E9%87%8F%E6%9C%89%E4%BB%80%E4%B9%88%E5%8C%BA%E5%88%AB](https://github.com/yangshun/front-end-interview-handbook/blob/master/Translations/Chinese/questions/javascript-questions.md#使用letvar和const创建变量有什么区别)) 



### bind,call,apply  作用及差异，实现 bind 函数

*  `call`和`apply`都用于调用函数，第一个参数将用作函数内 this 的值。然而，`call`接受逗号分隔的参数作为后面的参数，而`apply`接受一个参数数组作为后面的参数
* bind返回一个 **新** 函数，并且绑定了函数执行时候的 `this` 指向。同时，会把调用bind时除第一个参数外的所有参数，拼接到新的函数执行时获得的参数最前面

```javascript
if (!Function.prototype.bind) (function(){
  // Is this an error? We are invoking <call.bind> method before it's defined.
  var slice = Array.prototype.slice.call.bind(Array.prototype.slice);
  Function.prototype.bind = function() {
    var thatFunc = this, thatArg = arguments[0];
    var args = slice(arguments, 1);
    if (typeof thatFunc !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - ' +
             'what is trying to be bound is not callable');
    }
    return function(){
      var funcArgs = args.concat(slice(arguments))
      return thatFunc.apply(thatArg, funcArgs);
    };
  };
})();
```



参考 [bind polyfill](https://developer.mozilla.org/en-US/docs/web/javascript/reference/global_objects/function/bind#Polyfill) 



### 写一个 parseQuery 函数，把URL的参数解析为JSON对象

```javascript
function parseQuery(queryString) {
    var query = {};
    var pairs = (queryString[0] === '?' ? queryString.substr(1) : queryString).split('&');
    for (var i = 0; i < pairs.length; i++) {
        var pair = pairs[i].split('=');
        query[decodeURIComponent(pair[0])] = decodeURIComponent(pair[1] || '');
    }
    return query;
}
```



## React相关



### setState是同步的还是异步的？



* 生命周期和合成事件中：异步的，调用之后，**不能**从 `this.state`上读取到最新的值
* 异步代码和原生事件中：同步的，调用之后，会触发react更新，可以读取到最新的值

参考 [https://mp.weixin.qq.com/s/3jmJgZFktP2NMT8XLvdIKQ](https://mp.weixin.qq.com/s/3jmJgZFktP2NMT8XLvdIKQ) 



## 性能优化



### 优化页面性能需要考虑哪些方面？



1. 分析现状
   * 浏览器瀑布流、本地禁用缓存、打开缓存的加载时间
   * RUM统计用户真实的页面加载时间
2. 解决办法
   * 减少首次下载内容，保证最快让用户见到内容
   * 合理利用浏览器缓存，善于利用localstorage等存储介质
   * 网络连接时间长，避免请求数过多，css sprite，svg图标，图片 base64等
   * 域名收敛，静态资源统一域名，减少http请求头大小
   * 静态资源强缓存，cache-control，可以考虑 service-worker缓存
   * CDN，加速静态资源下载速度
   * 图片资源占据页面很大部分，需要减少图片大小，使用 webp 图片，不同dpr加载不同图片，svg、iconfont替换部分图标
   * lazyload非首屏资源，例如图片、JS
   * preload相关页面的资源
   * HTTP 2
3. 上线前后效果监测