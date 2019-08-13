# [日积跬步]系列之2019年

## 20190812

### Animated.Value 绑定的对应组件unmount时，动画会被停止

比如有 **一个** 加载中的旋转loading，会一直旋转，如果某一刻，这个组件被 `unmount` 了，对应的 `Animated.Value` 的动画会停止。可以参考这个 expo: [https://snack.expo.io/@sophister/animated-stop-on-unmount](https://snack.expo.io/@sophister/animated-stop-on-unmount)
 也可以参考这个 issue [https://github.com/facebook/react-native/issues/19826](https://github.com/facebook/react-native/issues/19826) ，里面有提到，当组件unmount的时候，会调用对应 `Animated.Value` 的停止方法

如果同一个 `Animated.Value`  被绑定到了 **多个** 组件上，那么在 **最后一个** 组件 `unmount` 的时候，会停止该 `Animated.Value` 的动画。可以参考这个 expo demo：[https://snack.expo.io/@sophister/animated-stop-multiple-unmount-test](https://snack.expo.io/@sophister/animated-stop-multiple-unmount-test)
