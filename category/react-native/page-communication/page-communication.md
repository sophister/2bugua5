# React Native 页面间通信

`React Native` (下文简称RN) 开发中，最近面临 **页面之间** 通信的问题，因为我们的APP，是属于 RN 和 native 混合的应用，所以这种页面间通信，又分为 RN页面间 、 RN和native间 的通信2种形式。

本文主要讨论，有2个页面 A 和 B，A在打开B之后，从B返回A页面时，需要返回一些数据 这种case。


## RN页面内部通信

在RN页面内部，我们用的是 `react-navigation` 作为导航，因此，RN页面内部的通信，主要先看看，`react-navigation` 有没有提供框架层面的支持。

1. 传递 `function` 作为 `param` ，然后在 B 页面里，调用父级A页面的回调函数

```javascript
//in screen A
this.props.navigation.navigate('Details', {
    param1: 86,
    otherParam: 'anything you want here',
    fun1: this.callbackFunction,
});
// in screen B
const fun1 = this.props.navigatin.getParam('fun1', null);
fun1()
```

2. 使用第三方状态管理，比如 `redux` 或 `mobx`；简单处理的话，应该也可以自己搞一个 `eventCenter` 来做


## RN 和 native通信




## 相关资料

* [How to let two screens communicate with each other?](https://github.com/react-navigation/react-navigation/issues/2840)
*
