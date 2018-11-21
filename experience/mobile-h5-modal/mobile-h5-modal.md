# 移动端H5弹窗处理

移动端H5弹窗，经常使用 `position: fixed` 来展示，但是移动端的弹窗，问题实在太多：


* 弹窗展示时，滑动屏幕，可以透过 `modal` 去滚动底部的DOM元素
* `iOS` 下对 `fixed`支持bug的问题
* 在弹窗里有 `input[type=text]` 时，弹出系统键盘，iOS渲染bug；
iOS下，`input`获取焦点时，光标莫名的移位；
如果弹窗高度过大，在软键盘弹出时，可能导致弹窗被部分遮挡；

本文希望通过一种很hack的方式，尝试解决上面的 **一部分问题** 。



## 阻止滚动弹窗后面的元素





> Unable to preventDefault inside passive event listener due to target being treated as passive.



相关文章：

* [Making touch scrolling fast by default](https://developers.google.com/web/updates/2017/01/scrolling-intervention)
* [Chrome 56 breaks touch events](https://github.com/facebook/react/issues/8968)
* [移动Web滚动性能优化: Passive event listeners](https://segmentfault.com/a/1190000007913386)


##