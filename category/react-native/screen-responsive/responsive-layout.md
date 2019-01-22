# React Native 屏幕适配

开发APP(包括H5)都面临一个问题，怎么适配各种尺寸的屏幕？

## H5方案：等比例缩放

在移动端的web页面适配上，我们采用的是手机淘宝出的 [lib-flexible](https://github.com/amfe/lib-flexible) 方案，大部分情况下，`rem` 一把梭，基本上解决了屏幕适配问题。

在 `React Native`的页面布局里，也很自然想到使用类似的方案，根据当前屏幕宽度，计算出应该显示的尺寸，梭哈即可。然鹅，在和UE同学确认时，才知道移动端的方案，他们并不太满意，换句话，UE在很多情况下，并不希望界面按照屏幕尺寸来等比缩放。


## RN 里怎么适配

和APP同学沟通之后，了解到了APP端之前的一些做法，再参考了一些业界方案，大体情况下，适配可以分成以下几种情况：

* 固定尺寸(按钮、文字大小、间距)
* 保持宽高比(比如banner图片)
* 间距固定，内容自适应(比如产品卡片宽度)
* 按屏幕等比缩放

再拿到UI设计稿后，对布局、尺寸自适应不够清楚的，直接和UE同学确认具体的适配方式，一般都对应到上述情况之一。

针对以前Android和iOS同学，使用的2套设计稿的情况(Android用1080，iOS用750)，统一使用 **750px的iPhone设计稿** ，设计同学只出一套设计稿，图片仍然给 `@2x` `@3x` 两套。

针对上述4种情况，统一提供几个工具函数，方便开发同学使用。开发同学拿到750的设计稿，量出某个尺寸，直接使用下面几个工具函数之一，来处理各种情况:

* `px2dp`: **不做** 等比例缩放，返回 `@1x` 的大小
* `scalePx2dp`: 根据当前 **屏幕宽度** 和750的比例，等比的计算当前屏幕下对应的大小

最后，针对上述几种适配情况，基本上这样来处理：

* 固定尺寸： 使用 `px2dp` 得到固定大小 (大多数尺寸都用固定尺寸)
* 保持宽高比：建议使用 [aspectRatio](https://facebook.github.io/react-native/docs/layout-props#aspectratio)
* 间距固定，内容自适应：使用 `flex` 布局
* 按屏幕等比缩放：使用 `scalePx2dp`

在实际布局中，大的块级布局，块级组件，**宽度** 应该尽量使用 `flex` 布局，避免 **宽度** 也使用 **固定尺寸** 来实现。

### aspectRatio 用法

在保持宽高比的情况下，我们应该尽可能使用 `aspectRatio` 来实现，比如，有个图片，在 750设计稿 里量来是 126*134 ，宽度占父级的 50%，我们可以这样来实现：

```javascript
const style = {
    width: '50%',
    aspectRatio: 126 / 134,
};
//later in render
<Image style={style}></Image>
```

### 两侧间距固定，中间自适应

比如，有个 `View` 组件，需要保持屏幕两侧间距为 20dp，内容自适应，我们可以这样实现：

```javascript
const style = {
    marginHorizontal: 20,
    alignSelf: 'stretch',
};
//render
<View style={style}></View>
```


## 相关资料

* [react-native-size-matters](https://blog.solutotlv.com/size-matters/)
* [REACT NATIVE SUPPORT MULTIPLE SCREEN SIZES](https://yaobin.me/blog/react-native-support-multiple-screen-sizes/)
* [https://github.com/nirsky/react-native-size-matters](https://github.com/nirsky/react-native-size-matters)
* [手机淘宝iphone适配协作模式](https://www.zhihu.com/question/25308946/answer/32240185)
