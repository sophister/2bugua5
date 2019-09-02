# [日积跬步]系列之2019年


## 20190902

`JavaScript` 中获取某个对象键的方法

* `Object.keys(obj)`: 返回 **可枚举** 的 **字符串** 键，不包含 `Symbol`类型的键
* `Object.getOwnPropertyNames(obj)`: 返回所有的 **字符串** 类型的键，不包含 `Symbol`类型的键
* `Object.getOwnPropertySymbols(obj)`: 返回所有 **Symbol** 类型的键
* `Reflect.ownKeys(obj)`: 返回所有的键，包括 **Symbol** 和 **字符串** 类型的，包括不可枚举的，等价于 `Object.getOwnPropertyNames(target).concat(Object.getOwnPropertySymbols(target))`


## 20190901

**递归拷贝某些后缀的文件到另外一个目录**

有两个目录，A和B：A目录下有很多文件，包括 `.html` `.js` `.css`，可能包含任意层级的子目录；B目录下是空的。

现在要拷贝A目录下所有的(包括子目录)的 **除了.html** 以外的其他文件到B目录，并且保持目录层级。

直接用 `cp` 命令，貌似很难保持目录层级关系，倒是可以使用通配符来匹配文件后缀。

最后发现用 `rsync` 命令可以很方便的实现，命令如下：

```shell
rsync -rv --exclude='*.html'  ./A/ ./B/
```


## 20190828

`Typescript`里定义 `async` 函数的类型

```javascript
// *错误* 的写法
interface WrongWay {
    func: async () => string;
}

// *正确*的写法
interface RightWay {
    func: () => Promise<string>;
}
```

只需要将函数的 **返回值** 定义为 `Promise`，这个函数就 **满足** 了 `async` 函数的要求。毕竟从结果上看，`async`函数和普通函数的区别，只是返回的是 `Promise`，在调用的时候可以加上 `await`，调用方并不关心函数是否是 `async` 的，只关心返回值是 `Promise` 就行了。

BTW，今天正好是到公司的4周年了😅

## 20190823

`React Native` (版本：0.59.3) 里， `View.measure(callback)` 拿到的结果是 `undefined`。但是 `View.onLayout` 可以拿到正确尺寸。要修复 `View.measure`返回 `undefined`的问题，有以下两个hack：

* 在 View 上设置 `onLayout`，比如空的占位函数 `onLayout={() => {}}`
* 在 View 上设置 `collapsable={false}`

根据 [collapsable](https://facebook.github.io/react-native/docs/view#collapsable)文档，猜测返回 `undefined` 的情况下，对应的 View 在native里被优化掉了……而添加上述的属性，应该都可以避免 View 被native优化掉吧


## 20190820

`String.prototype.repeat`

今天看之前native同学写的 node api接口，发现里面用到了 `string` 的一个 `repeat` 方法，返回将字符串重置N(N >= 0)次的一个新字符串。

```javascript
'abc'.repeat(-1);   // RangeError
'abc'.repeat(0);    // ''
'abc'.repeat(1);    // 'abc'
'abc'.repeat(2);    // 'abcabc'
'abc'.repeat(3.5);  // 'abcabcabc' (count will be converted to integer)
'abc'.repeat(1/0);  // RangeError
```

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/repeat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/repeat)


## 20190815

`React Hooks` 中 `useEffect` 和 `useLayoutEffect` 差别

根据官方文档，这两个hook的函数签名完全一样的，在绝大多数情况下加，官方推荐先试用 `useEffect`。

1. `useLayoutEffect` 的触发时机比 `useEffect` 早。才发现官方文档里，已经有说明了，`useLayoutEffect` 的触发时机，是对应 `class` 组件中的 `componentDidMount` 和 `componentDidUpdate` 的。`useLayoutEffect`是在浏览器绘制内容之前 **同步** 调用，因此，如果在 `useLayoutEffect` 里有很耗时的JS同步操作，那将阻塞浏览器的渲染。

2. 对于 `useEffect`，触发时机确实非常的晚，是在浏览器绘制内容之后，才会被调用。在 子组件 中的 `useEffect` 调用时机，甚至在 父组件 的 `componentDidMount`(或者 `componentDidUpdate`) 之后！ 可以看这个demo [https://codesandbox.io/s/useeffect-vs-uselayouteffect-nh8m2](https://codesandbox.io/s/useeffect-vs-uselayouteffect-nh8m2)

上面第2点，可能也是最近 `React Native`代码里，在 `useEffect` 里调用 `react-navigation`库的 `navigation.addListener` 首次回调不会触发的原因。

还有两篇文章可以参考下：

* [https://blog.logrocket.com/useeffect-vs-uselayouteffect/](https://blog.logrocket.com/useeffect-vs-uselayouteffect/)
* [https://kentcdodds.com/blog/useeffect-vs-uselayouteffect/](https://kentcdodds.com/blog/useeffect-vs-uselayouteffect/)


## 20190814

`React Native` 中使用 `gif` 动画图片

根据RN官方文档，Android下只是单独加一个包就可以了，加上之后，gif动画只执行了 **一次** ！但是在 `macOS`里预览图片时，是一直在重复执行动画的……找了半天，后来尝试把图片扔到chrome里，发现也不行，也只执行了一次动画。
猜想gif图片是不是有一些 *元数据* ，里面有字段来表示动画次数之类的，Google之后发现确实有个 `animation count`之类的字段。原来，这次UE给的gif图，在导出时没有设置动画次数，因此，在RN和chrome里只执行了一次动画


## 20190812

### Animated.Value 绑定的对应组件unmount时，动画会被停止

比如有 **一个** 加载中的旋转loading，会一直旋转，如果某一刻，这个组件被 `unmount` 了，对应的 `Animated.Value` 的动画会停止。可以参考这个 expo: [https://snack.expo.io/@sophister/animated-stop-on-unmount](https://snack.expo.io/@sophister/animated-stop-on-unmount)
 也可以参考这个 issue [https://github.com/facebook/react-native/issues/19826](https://github.com/facebook/react-native/issues/19826) ，里面有提到，当组件unmount的时候，会调用对应 `Animated.Value` 的停止方法

如果同一个 `Animated.Value`  被绑定到了 **多个** 组件上，那么在 **最后一个** 组件 `unmount` 的时候，会停止该 `Animated.Value` 的动画。可以参考这个 expo demo：[https://snack.expo.io/@sophister/animated-stop-multiple-unmount-test](https://snack.expo.io/@sophister/animated-stop-multiple-unmount-test)
