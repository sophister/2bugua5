# 一次奇怪的 Cookie 丢失，让我重新理解了 SameSite

>前言：相信很多人和我一样，知道cookie有个 same-site 属性，也大概知道3个值分别的含义。但是我猜**绝大部分同学，都不知道，chrome针对 Lax+跨域POST，还有个特殊的2分钟逻辑**，好久没踩到这种坑了，找了半天都没找到cookie是怎么“消失”的……



最近在调试一个跨域请求的时候，遇到一个挺诡异的现象。

我在 **A 域名** 的页面里，通过一个 HTML `<form>` 表单发起 `POST` 请求到 **B 域名**。
理论上，B 域名下之前已经保存了几个 session 级别的 cookie，按理说应该会被自动带上。
但实际情况是——**有时候会丢。**

而且更离谱的是：

* 用 **GET 请求** 的时候，cookie 都在；
* 用 **POST 请求**，部分 cookie 就不见了；
* 刷新几次后，甚至两个 session cookie 都会凭空消失，只剩一个。

一开始我以为是我自己逻辑写错了，后来我顺着浏览器的行为一点点排查，才发现罪魁祸首是那个看似不起眼的属性：**SameSite**。

---

## 一、SameSite 是什么？

`SameSite` 是 Cookie 的一个属性，用来限制跨站请求时浏览器是否应该携带 cookie。

为什么要限制？
因为这是防御 **CSRF（跨站请求伪造）** 攻击的一种机制。

Chrome、Firefox、Safari 等主流浏览器都已经默认启用了它。

---

## 二、SameSite 的三个取值

### 1. `SameSite=Strict`

最安全也是最严格的模式。

> 只有在同一个站点下的请求才会带上 cookie。
> 一旦跨域，就算是用户点击链接跳转，也不会带。

举个例子：

* 你在 `b.com` 登录成功，服务器返回：

  ```http
  Set-Cookie: sessionid=abc; SameSite=Strict
  ```
* 然后你在 `a.com` 上点了个链接跳到 `b.com`，
* 这时浏览器请求 `b.com` 时，不会带上 cookie。

适合用在一些**后台系统、管理面板**之类的高安全场景。

---

### 2. `SameSite=Lax`

相对“宽松”一点。

> 大多数同站请求会带上 cookie，部分跨站请求也会带，尤其是安全的 GET 导航请求。

也就是说：

* 用户点击链接跳转（GET） ✅ 会带；
* 表单 POST、iframe、fetch 这种跨站请求 ❌ 不会带。

举个例子：

```text
a.com → b.com (GET 导航)：✅ cookie 携带
a.com → b.com (POST 表单)：❌ cookie 不携带
```

---

### 3. `SameSite=None`

这个值的意思是：我就是要跨站携带 cookie。
但要注意一点：

> 必须同时加上 `Secure`，也就是说只能在 HTTPS 下生效。

```http
Set-Cookie: sessionid=abc; SameSite=None; Secure
```

这在很多场景都必须要用，比如：

* 第三方登录（OAuth）
* 嵌入式 iframe
* 跨域 API 调用

---

## 三、如果不写 SameSite，会怎样？

这才是我这次踩坑的关键点。

以前（Chrome 80 之前，未考证），如果你不写 SameSite，浏览器默认就是 **无约束的跨域 cookie**。
但现在（Chrome 80+，未考证），默认值已经变成：

```text
SameSite=Lax
```

这意味着：

* 你的 cookie 默认**不支持跨站 POST**；
* 只有 GET 导航（比如点击链接）才会携带。

所以，如果你的业务逻辑依赖“表单跨域 POST 携带登录态”，恭喜——你已经掉进坑里了。

---

## 四、Chrome 的“Lax + POST 2分钟例外”

更坑的是，Chrome 还为了兼容老系统，**特地加了个 2 分钟的特殊规则**（官方称之为 *Lax + POST Temporary Exception*）：

> 如果一个 cookie 没有显式声明 SameSite 属性，
> 那么它在被设置后的 2 分钟内，跨站 POST 请求仍然可以携带，
> 之后就不行了。

也就是说：

1. 你刚登录（Set-Cookie 发回来了）
2. 马上在另一个站点发起跨域 POST
3. ✅ Cookie 会带上
4. 两分钟后再试
5. ❌ Cookie 不带了

这就解释了为什么我在调试接口时会看到“有时候带 cookie、有时候不带”的情况。
不是浏览器抽风，而是 Chrome 按（自己的）规范执行。

---

## 五、结合我的实际问题

我在项目中是这样的：

* 在 A 域名的前端，用表单 POST 提交到 B 域名；
* B 域名上保存了几个 session 级别 cookie；
* 服务端没显式设置 SameSite；
* 结果表现是：刚登录完马上请求还行，隔一会儿 cookie 就“失踪”了。

最终复盘下来：

1. 未显式设置 SameSite → 浏览器默认 Lax；
2. Lax 不支持跨站 POST；
3. Chrome 2分钟特例 → 刚开始“假象”正常；
4. 超过 2 分钟 → 全部按规范丢弃。

真相就是这么“合理又坑”。

---

## 六、怎么解决？

最根本的办法是：**不要让浏览器替你猜。**

明确告诉它你要什么样的行为。

如果需要跨站请求携带 cookie，就要：

```http
Set-Cookie: sessionid=abc; SameSite=None; Secure
```

如果只在自己域下使用：

```http
Set-Cookie: sessionid=abc; SameSite=Strict
```

同时确保：

* 一定要走 HTTPS；
* 不依赖 Chrome 的临时兼容行为；
* 尤其在跨域表单提交、iframe 内调用接口、OAuth 登录这些场景，必须显式设置。

---

## 七、总结一下

| SameSite | 跨站 GET | 跨站 POST | 是否要求 HTTPS | 适用场景       |
| -------- | ------ | ------- | ---------- | ---------- |
| Strict   | ❌      | ❌       | 否          | 后台、敏感操作    |
| Lax（默认）  | ✅      | ❌       | 否          | 普通站点登录态    |
| None     | ✅      | ✅       | ✅          | 第三方登录、跨域组件 |

Chrome 对 cookie 的安全策略变得越来越严格，这是趋势。
对我们前端来说，理解这些细节，能避免很多“表面看不出问题但调半天”的坑。

尤其是这种 **“偶尔丢 cookie”** 的情况，大多数时候不是代码问题，而是规范变了。

---

**一句话结尾：**

> 不写 SameSite，等于让浏览器替你做决定。
> 而浏览器从不保证你的业务逻辑不被“保护”掉。

## 相关文档

- [https://www.chromium.org/updates/same-site/?utm_source=chatgpt.com#nov-1-2019](https://www.chromium.org/updates/same-site/?utm_source=chatgpt.com#nov-1-2019)
- [https://web.dev/articles/samesite-cookies-explained](https://web.dev/articles/samesite-cookies-explained)
- [https://groups.google.com/a/chromium.org/g/blink-dev/c/AknSSyQTGYs/m/YKBxPCScCwAJ](https://groups.google.com/a/chromium.org/g/blink-dev/c/AknSSyQTGYs/m/YKBxPCScCwAJ)