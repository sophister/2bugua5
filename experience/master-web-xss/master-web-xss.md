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

看到上面这个例子，嗯，所处的 上下文(`context`)是 `javascript环境`，那就对 `<>'"=` 编码为 **Unicode转义序列** 就搞定了嘛。还是 **too young too simple**，没有考虑到这是一个 **多维度环境** 。在 **JS 解释器解码** 之后，执行代码时，因为需要一个 `URL`，所以 **URL解释器** 会介入，还会进行1次 **URL解码** 。解码顺序为：1. JS解码；2. URL解码 。

## URL解码

看如下例子：

```html
<a href="javascript:location.href='http%3A%2F%2Fwww.baidu.com'">test1</a>
<a href="http%3A%2F%2Fwww.baidu.com">test2</a>
<a href="http://%77%77%77%2e%62%61%69%64%75%2e%63%6f%6d">url encode2, keep :// </a>
```

例子里的前面2个case，`test1` 可以正常跳转；`test2` 却不行，折磨了好几天，今天终于看到 [这篇文章](https://www.attacker-domain.com/2013/04/deep-dive-into-browser-parsing-and-xss.html) ，算是基本想明白了。


## 相关资料

* 这篇讲解大概的编码规则，可以大概扫一眼  [http://blog.csdn.net/raintungli/article/details/51276322](http://blog.csdn.net/raintungli/article/details/51276322)
* 这篇文章讲的不错，但是有些细节不清楚  [http://imweb.io/topic/57024e4606f2400432c1396d](http://imweb.io/topic/57024e4606f2400432c1396d)
* 这篇大概讲浏览器如何解码，可以看一下  [http://xuelinf.github.io/2016/05/18/%E7%BC%96%E7%A0%81%E4%B8%8E%E8%A7%A3%E7%A0%81-%E6%B5%8F%E8%A7%88%E5%99%A8%E5%81%9A%E4%BA%86%E4%BB%80%E4%B9%88/](http://xuelinf.github.io/2016/05/18/%E7%BC%96%E7%A0%81%E4%B8%8E%E8%A7%A3%E7%A0%81-%E6%B5%8F%E8%A7%88%E5%99%A8%E5%81%9A%E4%BA%86%E4%BB%80%E4%B9%88/)
* 宜人贷的这篇博客讲的更深入一些 [https://security.yirendai.com/news/share/26](https://security.yirendai.com/news/share/26)
* 这篇文章强烈推荐仔细阅读，应该囊括了前面两篇的知识  [https://www.attacker-domain.com/2013/04/deep-dive-into-browser-parsing-and-xss.html](https://www.attacker-domain.com/2013/04/deep-dive-into-browser-parsing-and-xss.html)
* 浏览器URL解析的W3C规范，只看懂了一小部分，[https://url.spec.whatwg.org/#url-parsing](https://url.spec.whatwg.org/#url-parsing)
