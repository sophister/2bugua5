# [日积跬步]系列之2019年

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
