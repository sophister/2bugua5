# XSS 重新入门

`XSS`，**Cross-site scripting**，也算是 **web安全** 一个老生常谈的问题了。

简单来说，导致 `XSS` 的原因，就是 **来自client的输入，没有进行适当的转义、encode，导致client输入的内容，被浏览器当做普通html/script来解释、执行** 。

## HTML实体编码

举个栗子：不少网站都会提供搜索功能，允许用户输入query，然后在搜索结果页，展示用户输入的query。在展示的时候，如果处理不当，就很容易被 `XSS`攻击。设想搜索结果页的模板是这样的：

```html
<div>
    您搜索的 {{ query }} 一共包含N条结果
</div>
```

用户输入一般的检索词，很完美。但如果用户输入的query是这段代码呢：`<img src=# onerror=alert(1) />` ，最终搜索结果页的html结构是这样的：

```html
<div>
    您搜索的 <img src=# onerror=alert(1) /> 一共包含N条结果
</div>
```

进入该页面，自动弹出 `alert`框，bingo，恭喜被 `XSS`了！

可能会感觉这个很简单，直接把 `query` 进行 `HTML实体编码` 不就行了么。那再看看下面这个例子：

## JS Unicode转义序列

```html
<script>
    // userInputLink 是来自用户的输入
    location.href = '{{ userInputLink }}';
</script>
```

看到上面这个例子，嗯，所处的 上下文(`context`)是 `javascript环境`，那就对 `<>'"=` 编码为 **Unicode转义序列** 就搞定了嘛。还是 **too young too simple**，没有考虑到这是一个 **多维度环境** 。在 **JS 解释器解码** 之后，执行代码时，因为需要一个 `URL`，所以 **URL解释器** 会介入，还会进行1次 **URL解码** 。解码顺序为：1. JS解码；2. URL解码 。其实，理论上，这里具体的解码个数是 **不确定** 的，因为 用户可能输入的是 `javascript:alert(document.cookie)` 这样的，在执行的时候，最后还会进行一次 JS解码！！

## URL解码

看如下例子：

```html
<a href="javascript:location.href='http%3A%2F%2Fwww.baidu.com'">test1</a>
<a href="http%3A%2F%2Fwww.baidu.com">test2</a>
<a href="http://%77%77%77%2e%62%61%69%64%75%2e%63%6f%6d">url encode2, keep :// </a>
```

例子里的前面2个case，`test1` 可以正常跳转；`test2` 却不行，折磨了好几天，今天终于看到 [这篇文章](https://www.attacker-domain.com/2013/04/deep-dive-into-browser-parsing-and-xss.html) ，算是基本想明白了。

第1个例子中，解码顺序为：`HTML解码` -> `URL解码` -> `JS解码` -> `URL解码`。因为没有 `HTML实体`，`HTML解码`之后没变化；接下来，是在 `URL上下文`，会进行 `URL解码`，在这一步，能够正确识别出 `javascript:` 这个 `protocol`，证明是合法的 `URL`，因此，会继续对剩余部分进行 `URL解码`，将后面的 `location.href='http%3A%2F%2Fwww.baidu.com'` 解码之后得到 `location.href='http://www.baidu.com'` ；当用户点击链接的时候，发现是执行JS，因此会进行 `JS解码`；执行JS时，又是一个 `URL上下文`，因此，会 **再次进行 URL解码**，这一步其实不会有什么改动，顺利跳转到百度。

再来看第2个例子，解码顺序为：`HTML解码` -> `URL解码`。在 `HTML解码`中，同样不会有什么改动；接下来进行 `URL解码`，这次因为 `://` 被编码了，`URL parser`不能识别出正确的URL，因此不会跳转到百度。

同理，可以解释第3个例子，为什么能成功跳转。


## Content Security Policy (CSP)

要比较完美的解决 `XSS` 问题，在根据不同的 上下文(context) 进行相应转义、encode之后，最好再配合 `[CSP](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CSP)` 这个最后一道防火墙。

MDN上对`CSP`要解决的问题，是这样描述的：

> CSP 的主要目标是减少和报告 XSS 攻击 ，XSS 攻击利用了浏览器对于从服务器所获取的内容的信任。恶意脚本在受害者的浏览器中得以运行，因为浏览器信任其内容来源，即使有的时候这些脚本并非来自于它本该来的地方。
>
CSP通过指定有效域——即浏览器认可的可执行脚本的有效来源——使服务器管理者有能力减少或消除XSS攻击所依赖的载体。一个CSP兼容的浏览器将会仅执行从白名单域获取到的脚本文件，忽略所有的其他脚本 (包括内联脚本和HTML的事件处理属性)。



## 完整demo

可以在这个demo中，测试各种组合情况： [http://output.jsbin.com/jujovar](http://output.jsbin.com/jujovar) 。


## 相关资料

* 这篇讲解大概的编码规则，可以大概扫一眼  [http://blog.csdn.net/raintungli/article/details/51276322](http://blog.csdn.net/raintungli/article/details/51276322)
* 这篇文章讲的不错，但是有些细节不清楚  [http://imweb.io/topic/57024e4606f2400432c1396d](http://imweb.io/topic/57024e4606f2400432c1396d)
* 这篇大概讲浏览器如何解码，可以看一下  [http://xuelinf.github.io/2016/05/18/%E7%BC%96%E7%A0%81%E4%B8%8E%E8%A7%A3%E7%A0%81-%E6%B5%8F%E8%A7%88%E5%99%A8%E5%81%9A%E4%BA%86%E4%BB%80%E4%B9%88/](http://xuelinf.github.io/2016/05/18/%E7%BC%96%E7%A0%81%E4%B8%8E%E8%A7%A3%E7%A0%81-%E6%B5%8F%E8%A7%88%E5%99%A8%E5%81%9A%E4%BA%86%E4%BB%80%E4%B9%88/)
* 宜人贷的这篇博客讲的更深入一些 [https://security.yirendai.com/news/share/26](https://security.yirendai.com/news/share/26)
* 这篇文章强烈推荐仔细阅读，应该囊括了前面两篇的知识  [https://www.attacker-domain.com/2013/04/deep-dive-into-browser-parsing-and-xss.html](https://www.attacker-domain.com/2013/04/deep-dive-into-browser-parsing-and-xss.html)
* 浏览器URL解析的W3C规范，只看懂了一小部分，[https://url.spec.whatwg.org/#url-parsing](https://url.spec.whatwg.org/#url-parsing)
