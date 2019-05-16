# React Native 动画



## Animated 基本用法



RN里使用 `Animated` 来进行动画相关操作，比如UI组件 `Animated.View` `Animated.Image` `Animated.Text` `Animated.ScrollView` 。使用动画，需要依赖这两个动画相关的值：`Animated.Value` `Animated.ValueXY` 。

基本的开启和结束一个动画，如下：

```javascript
const fadeAnim = new Animated.Value(0),  // Initial value for opacity: 0
Animated.timing(                  // Animate over time
      fadeAnim,            // The animated value to drive
      {
        toValue: 1,                   // Animate to opacity: 1 (opaque)
        duration: 10000,              // 动画时长
      }
).start( animationEnd );                        // 启动动画，可以传入一个动画结束的回调函数

//动画执行回调，动画正常执行完和主动结束，都会调用这个。
//通过 result.finished === true 来判断是否正常执行完动画
function animationEnd(result){
  if(result.finished){
    // 动画正常执行完
  }
}

// 可以主动停止一个动画，同样会调用 animationEnd
fadeAnim.stopAnimation()；

```

**插值**，有的时候，我们会将原始的动画值，映射成其他的值，就需要用到动画插值，比如这样:

```javascript
const value = new Animanited.Value(0);
const style = {
  color: 'green',
  opacity: value.interpolate({
    inputRange: [0, 0.5, 1],
    outputRange: [0, 1, 0],
  }),
  transform: [
    {
      translateY: value.interpolate({
        inputRange: [0, 0.5, 1],
        outputRange: [0, 150, 0],
      }),
    },
  ],
};

// render
<Animated.Text style={style}>some text</Animated.Text>
```







## 相关资料



* <https://github.com/oblador/react-native-animatable> 
* 