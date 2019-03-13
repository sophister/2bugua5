# PWA H5页面性能优化

`PWA` 这个名词出来有一段时间了，一直没能实践下，业务上暂时也还没什么动机能上的。但是越来越多的大厂，包括 饿了么、Twitter、阿里等，都开始在生产环境大量使用相关技术，是时候重新来看看这个东东了。

不过，`PWA`主要是给 `SPA` 来使用的，我们目前的业务确实还用不上。那就用一下其中的 `Service Worker`这个东东吧，试试对我们的 `H5` 秒开有没有什么功效。根据之前测试的情况，目前我们APP里的 `WebView`，处理缓存是存在问题的，理论上我们的静态资源都是强缓存的，但实际上发现，很多时候，页面仍然会重新请求静态资源，导致H5加载速度比较慢。如果能使用 `Service Worker`技术，**主动** 地缓存静态资源，而不是被动地交给浏览器处理，是不是能够解决静态资源的缓存问题呢？


## service worker

`Service Worker` (下文简称SW)，能够拦截作用域(scope)下的页面，以及该页面发出的请求，包括 **跨越请求** 。

### 浏览器支持情况

根据 [https://caniuse.com/#feat=serviceworkers](https://caniuse.com/#feat=serviceworkers) 的数据，`android 5` 之后，`WebView` 就支持 `serviceworker`；`iOS 11.3` 之后，也支持了。

找到这个MDN的链接 [https://mdn.github.io/sw-test/](https://mdn.github.io/sw-test/) ，来测试 `serviceworker` 的缓存情况。

经过实际的测试，`iOS 11.3`上，`Safari`是支持 `serviceworker`的，在 `iphone simulator`里，通过添加 `com.apple.developer.WebKit.ServiceWorkers  YES` ，`WKWebView`也可以支持 `serviceworker`，但是！但是，`iOS`的 **APP里 WKWebView** 是不支持 `serviceworker` 的！！库克这个糟老头子，实在是坑啊……

### ServiceWorkerGlobalScope

SW 的JS是在单独的进程这执行的，有自己的全局作用域，也有自己的全局变量(状态)。但是，根据 [https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerGlobalScope) 的说明，SW 的去全局状态，当SW被中断或者重启时，会被重置为默认状态，**不会** 持久化。因此，我们 **不应该** 在SW中设置自定义的全局变量，因为这些变量的状态，并没有持久化。

### 事件

* install: 初次安装SW时，在SW内触发的事件，可以在这个事件里，添加一些静态资源的缓存
* activate: SW安装成功之后，会触发这个事件，可以在这里，删除旧版本的缓存
* fetch: 在控制的页面内，发起请求时，触发SW这个事件，可以替换返回的响应
* 更新: 当发现新的SW时，将会加载并执行该SW，触发 `install` 事件，但 **不会** 触发 `activate` 事件。如果当前有激活的SW，新的SW将会等待旧的退出(刷新页面没用，必须要关闭旧的SW控制的所有标签页)，再触发新的 `activate` 事件并接管页面控制。也可以在 `install` 事件中，调用 `self.skipWaiting()` 来跳过等待状态，尽快激活自己

### 作用时机

* 首次注册&激活：在这种情况下，SW **不会** 拦截到页面内的请求，即使在SW激活之后，页面后续产生的请求，都 **不能** 拦截到。除非 SW 主动调用 `self.clients.claim()` ，调用之后，即使是首次注册激活，也能拦截页面后续的请求。见 [https://developer.mozilla.org/en-US/docs/Web/API/Clients/claim](https://developer.mozilla.org/en-US/docs/Web/API/Clients/claim)
* 注销：在调用 `unregister` 之后，当前激活的SW **仍然** 会控制当前页面的请求，直到页面关闭

### SW和主页面通信

1. 在主页面里，向SW触发事件，可以像下面这样:

```javascript
// in app.js
navigator.serviceWorker.controller.postMessage({
    name: 'hello',
    age: 30,
    deep: {
        arr: [ 1, 2, 3],
        test: true,
    }
});
// in sw.js
self.addEventListener('message', function (event) {
    console.log(event.data);
});
```

2. 在 SW 里向主页面触发事件：

```javascript
// in sw.js
self.clients.matchAll().then(function (clients) {
    console.log('clients in sw: ', clients);
    clients.forEach(function (client) {
        client.postMessage({
            hello: "word",
            yes: true,
            nest: {
                obj: {age: 31},
                num: -23,
                bool: false,
            },
        });
    });
});
// in app.js
navigator.serviceWorker.addEventListener('message', function(event){
    console.log('message from sw: ', event.data);
});
```


## 跨域请求

### Request.mode

根据 [https://developer.mozilla.org/en-US/docs/Web/API/Request/mode](https://developer.mozilla.org/en-US/docs/Web/API/Request/mode) 文档的介绍，跨域请求中，需要特别关注 `Request.mode` 这个 **readonly** 属性，这个属性会决定，在 `Response` 中，我们可以读取到哪些属性。`Request.mode` 包括以下几个值： `same-origin` `no-cors` `cors` `navigate` 。在不同的情况下，`Request.mode`的默认值是不一样的，重点要关注 `no-cors` 和 `cors`

* `no-cors`: 只允许发起 `HEAD` `GET` `POST` 请求，JS **不能** 读取到 `Response` 中的任何属性。`html`页面内嵌的资源请求，不设置 `crossorigin` 的情况下，默认都是 `no-cors` 模式，包括这些标签：`<link>` `<script>` `<img>` `<audio>` `<video>` `<object>` `<embed>` `<iframe>`。
* `cors`: 遵循 `CORS` 协议，并且允许在 JS 里读取 `Response` 的部分属性

### Response.type

对应上述的 `Request.mode`，还存在一个只读的 `Response.type` 属性，用来判断当前的响应，是一个什么类型的请求，包括这些值：

* `basic`: 正常的、满足 `同源策略` 的请求，允许JS读取 除了 `Set-Cookie` 和 `Set-Cookie2` 以外的，所有响应header
* `cors`: 合法的跨域请求，能读取部分响应header，包括http状态码
* `error`:
* `opaque`: 对应 `Request.mode` 是 `no-cors` 的请求，**不能** 读取任何属性，包括 http状态码
* `opaqueredirect`:  

### CORS crossorigin

通常情况下，我们页面里的静态资源，包括 `<img>` `<link>` `<script>` 都会通过单独的域名来加载，这个域名一般和主页面的域名 **不同**，在 `html5` 里允许我们设置元素的 `crossorigin` 属性，来主动控制是否发送用户的credentials，包括 `Cookie` 。`crossorigin` 在默认情况下(即元素上不设置该属性)，不会发起 `CORS` 请求；该属性只有2个可能的值：

* `anonymous`: 发起 `CORS` 请求，并且将 credentials 设置为 `same-origin`
* `use-credentials`: 发起 `CORS` 请求，并且将  credentials 设置为 `include`

### service worker 拦截跨域请求

`serviceworker`可以拦截作用域下发起的所有请求，包括 **跨越请求** 。通常在缓存静态资源的时候，我们需要确保资源是正确响应的(比如返回http状态码200)，如果返回的是 `404` 或者 `500` 之类的，那么就不应该缓存该资源。但是在 **跨域** 的请求下，默认的 `Request.mode no-cors` 我们 **不能** 读取返回的状态码，也就不知道资源是否OK；因此需要设置跨域资源的 `cors` 属性，比如页面里静态资源，设置 `crossorigin=anonymous` ，这样会触发浏览器发起 `CORS` 请求，也就要求我们的服务器返回header中，增加 `Access-Control-Allow-Origin` 来允许我们跨域访问该资源。

## 上线方案

针对我厂的情况，在移动端H5页面，我们主域名是 `m.renrendai.com`，引用的JS、css、图片等静态资源，使用的是 `s0.renrendai.com`，存在静态资源跨域的情况，因此，需要以下准备工作：

* 修改线上域名 `s0.renrendai.com` 下的 `nginx` 配置，配置 `Access-Control-Allow-Origin` 允许 `m.renrendai.com` 跨域请求
* 修改模板、JS组件，加载 `<script>` `<link>` `<img>` 时，附带上 `corssorigin=anonymous` 属性
* 使用 `Workbox` 库
* **不** 缓存页面
* 在 `serviceworker` 代码里，只缓存 `s0.renrendai.com` 和 `m.renrendai.com` 下的静态资源(包括JS、CSS、图片)。需要确认，在缓存 `s0.renrendai.com` 的资源时，只缓存http状态码是 `200` 的
* 静态资源(JS、CSS、图片)，因为文件名都带了 `md5`，因此全部使用 **cache first** 策略
* 灰度&回滚：增加可动态修改的开关，用来开启&关闭 `serviceworker` 。

**PS**： 考虑到后期可能会上线 PC 端的对应缓存方案，针对 `s0.renrendai.com` 的 `nginx` 配置修改，可能要动态的根据请求 `Origin` 来返回不同的 `Access-Control-Allow-Origin` 值。


## 相关资料

* [让老板虎躯一震的前端技术，KPI杀手](https://zhuanlan.zhihu.com/p/55072221?utm_source=wechat_session&utm_medium=social&utm_oi=827877323658395648)
* [腾讯X5浏览器内核-Service Worker最佳实践](https://x5.tencent.com/tbs/guide/serviceworker.html)
* [PWA 在饿了么的实践经验](https://zhuanlan.zhihu.com/p/25800461)
* [lavas-service worker](https://lavas.baidu.com/pwa/offline-and-cache-loading/service-worker/service-worker-introduction)
* [谨慎处理 Service Worker 的更新](https://zhuanlan.zhihu.com/p/51118741)
* [service-workers-unavailable-in-wkwebview-in-ios-11-3](https://stackoverflow.com/questions/49673399/service-workers-unavailable-in-wkwebview-in-ios-11-3)
* [what-limitations-apply-to-opaque-responses](https://stackoverflow.com/questions/39109789/what-limitations-apply-to-opaque-responses)
* [Workbox Google开源的service worker工具库](https://developers.google.com/web/tools/workbox/)
* [Workbox: Handle Third Party Requests](https://developers.google.com/web/tools/workbox/guides/handle-third-party-requests)
* [PWA系列 -- 分享PWA在阿里体系内的实践经验](https://zhuanlan.zhihu.com/p/50502316)
* [How do I uninstall a Service Worker?](https://stackoverflow.com/questions/33704791/how-do-i-uninstall-a-service-worker)
* [淘宝FED: Workbox 3：Service Worker 可以如此简单](http://taobaofed.org/blog/2018/08/08/workbox3/)
