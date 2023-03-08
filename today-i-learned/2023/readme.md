# [日积跬步]系列之2023年



## 20230308 window.innerWidth不可靠



最近在一个*SSR*项目里，遇到一个奇怪的问题，`window.innerWidth` 并不是真实的浏览器可视区域宽度，而是会受 **DOM结构里子元素width** 的影响（BTW，在桌面Chrome里没问题，但是在移动端Chrome里有问题）。



于是写了下面这个demo，来验证 `window.innerWidth`是否真的不可靠。

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>window.innerWidth affected by dom's width</title>
  <script>
    !function(){
      var msg = 'run in <head>\n window.innerWidth=' + window.innerWidth;
      msg += '\ndocument.documentElement.clientWidth=' + document.documentElement.clientWidth;
      alert(msg);
    }();
  </script>
</head>
<body>

  <div style="width: 1000px;" class="con">
    <div>this green bordered div width is <span class="width-con">1000px</span> width</div>
    <button id="btn">get width</button>
    <button id="plus">add div width by 100px</button>
  </div>
  <div>aaa</div>
  <div>bb</div>
  <div>cc</div>
  <div>ddd</div>
</body>
</html>
```

忽略了CSS代码，下面是JS代码。很简单，初始状态，我们会给 `.con` 一个宽度是 `1000px`，然后可以获取 `window.innerWidth` 和 `document.documentElement.clientWidth` ，之后再增加 `.con` 的宽度，再次获取：

```javascript
var btn = document.getElementById('btn');

btn.addEventListener('click', function(){
  var msg = 'window.innerWidth=' + window.innerWidth;
  msg += '\ndocument.documentElement.clientWidth=' + document.documentElement.clientWidth;
  alert(msg);
}, false);


var initWidth = 1000;
var con = document.querySelector('.con');
var plusBtn = document.getElementById('plus');
var widthCon = document.querySelector('.con .width-con');

plusBtn.addEventListener('click', function(){
  initWidth += 100;
  con.style.width = initWidth + 'px';
  widthCon.innerHTML = con.style.width;
}, false);
```

上面代码，在PC端Chrome执行是没有问题的。下面是在移动端Chrome的执行截图：

1. **在 `<head>` 里运行结果** 

   ![init value in head](./assets/20230308/head.jpeg)

2. **`div width=1000px` 读取结果**

   ![width=1000px](./assets/20230308/width1000.jpeg)

3. **`div width=1300px` 读取结果**

   ![width=1300px](./assets/20230308/width1300.jpeg)

4. **`width=2400px` 读取结果**

   ![width=2400px](./assets/20230308/width2400.jpeg)



你也可以通过这个demo链接访问试试：[https://output.jsbin.com/wokabot](https://output.jsbin.com/wokabot) 



说明：

* 上述截图是在 Android的Chrome里截图，在我手机上，微信里截图也这样。显然，`window.innerWidth` 的值收到了页面内部元素 `width` 的影响，但是看上去 `window.innerWidth` 有个最大值，内部元素 `width` 超过一定宽度，`window.innerWidth` 不会继续增大。
* 找了其他大佬的iOS里，`window.innerWidth` 值没有问题，和PC端一样，正确的
* PC端的Chrome里是 **正确** 的，没问题



想起之前移动端H5适配方案（基于 `rem` 方案），会用到浏览器宽度来计算 `<html>` 的 `font-size`， 赶紧看看阿里官方代码（[https://github.com/amfe/lib-flexible/blob/2.0/index.js#L18](https://github.com/amfe/lib-flexible/blob/2.0/index.js#L18)），还真是没有用 `window.innerWidth`，而是用的 `document.documentElement.clientWidth` ，有可能也是遇到过类似问题？



一句话结论：**在获取浏览器可视区域宽度的地方，应该使用 `document.documentElement.clientWidth` ，不要使用 `window.innerWidth` ** 。

