# 【译】URL 即状态

>URL在web开发中，算是一个非常基础的概念，我见过许多开发（特别是后端，并不是针对谁），连URL基本的一些知识都缺乏，最多的问题，就是在手动拼接URL的时候，`key=value`里经常忘记了 `URLEncode`。这篇文章虽然没有涉及到这部分，但是也比较清楚的讲解了URL的基本知识，以及在web前端领域的一些常见用法，时不时看看，还是有些收获。

>原文链接：
>https://alfy.blog/2025/10/31/your-url-is-your-state.html


最近几周我在写《URL 设计的隐藏成本》那篇文章时，需要在文章中添加 SQL 语法高亮。我去 PrismJS 网站找配置选项，但下载页上的各种选项把我弄得眼花缭乱。回到代码里我注意到文件顶部有一段注释包含一个 URL：

```
https://prismjs.com/download.html#themes=prism&languages=markup+css+clike+javascript+bash+css-extras+markdown+scss+sql&plugins=line-highlight+line-numbers+autolinker 
```

我已经完全忘了这段 URL 的存在，但点开后看到下载页里所有的复选框、下拉框和选项 **都预先被选中并反映出我当时的配置**：主题、语言、插件全都匹配。这个 URL 从一个简单的链接变成了完整的配置状态编码。就这样，一个 URL 完整重构了我当时的 UI 状态——无需数据库、cookie、localStorage，只有一个 URL。

这让我意识到，我们作为前端工程师，往往忽视了 URL 作为状态管理工具的潜力。我们习惯借助全局状态仓库、Context、缓存等各种抽象去管理状态，却往往忽略了 Web 最古老、最优雅的状态机制之一：**URL

在上一篇文章里我谈到了糟糕的 URL 设计带来的隐藏成本。今天我想从另一个角度出发，讨论 URL 的巨大价值——**如何把 URL 当作一等公民的状态容器来设计现代 Web 应用。**

---

## 被忽视的 URL 力量

Scott Hanselman 曾说过 “URL 就是 UI”，这句话非常有洞见。URL 不仅仅是浏览器获取资源的技术地址，它本身也是用户界面的一部分。

但 URL 不止是 UI，它还是状态容器。每当你构建一个 URL 时，其实就是在决定哪些信息需要保留、哪些需要共享、哪些可以被书签保存。URL 为我们免费提供了这些能力：

* **可分享性**：把链接发给别人，他们看到的状态与你一致
* **可收藏性**：保存一个 URL 即保存一次状态快照
* **浏览器历史**：前进/后退按钮自动有效
* **深度链接**：能够直接跳转到应用内部的特定状态

URL 让 Web 应用变得可预测而可靠。它们是 Web 最原始的状态管理方案，自 1991 年起一直稳定可用。问题不是 URL 是否能够保存状态，而是我们是否已经充分利用了它。

>译注：作者这里提到的**可收藏性**，就是指浏览器书签，在浏览器扩展还没有兴起的时候，借助书签来实现类似扩展功能，还是挺有意思。这类书签的协议，不再是 `https://`，而是 `javascript:`，URL内容就是一段 `javascript`代码，在某个H5页面点击的时候，就会执行这段JS，在目标H5页面插入JS来执行。

---

## URL 如何编码状态

URL 不同部分适合表达不同类型的状态信息：

### 1. 路径段（Path Segments）

用于层级式的资源导航，例如：

```
/users/123/posts         // 用户 123 的帖子
/docs/api/authentication // 文档结构
/dashboard/analytics     // 应用模块界面
```

这种结构表达了自然的资源层级关系。

### 2. 查询参数（Query Parameters）

适合表达过滤条件、选项、配置等，例如：

```
?theme=dark&lang=en      // 主题和语言
?page=2&limit=20         // 分页
?status=active&sort=date // 过滤与排序
?from=2025-01-01&to=2025-12-31 // 日期范围
```

查询参数最适合保存可变的 UI 状态。

### 3. 片段（Fragment / Hash）

用来进行页面内导航或定位，例如：

```
#L20-L35     // GitHub 行高亮
#features    // 滚动到特定章节
#/dashboard  // 老 SPA 路由用例
```

片段部分通常用于客户端导航，它不会被服务器发送。

此外，Text Fragments 还能定位页面上的具体文字，这让链接更加精确但不常用。

---

## URL 状态的现实例子

如下是一些真实场景中，URL 承载状态的例子：

### PrismJS 配置

```
https://prismjs.com/download.html#themes=prism&languages=markup+css+clike+javascript&plugins=line-numbers
```

整个语法高亮器的配置信息都被编码在 URL 里。改变 UI 配置，URL 也随之改变；分享 URL，别人即可获得完全相同配置。

### GitHub 行高亮

```
https://github.com/.../file.m#L108-L136
```

定位到特定文件，同时还高亮特定行号。点击链接就能准确看到讨论的代码片段。

### Google 地图视图

```
https://www.google.com/maps/@22.443842,-74.220744,19z
```

经纬度与缩放级别全部编码在 URL 里，分享后他人看到的地图视角完全一致。

### 设计工具状态

```
https://www.figma.com/file/xxx?node-id=123:456&viewport=100,200,0.5
```

这类链接不仅定位到具体设计文件，还包括画布视角和选中状态，是协作式工具的重要体验。

### 电商筛选

```
https://store.com/laptops?brand=dell+hp&price=500-1500&rating=4&sort=price-asc
```

每次过滤、排序、价格区间全部保存在 URL 里。用户可以收藏、分享、刷新状态不丢失。

---

## 什么状态适合放进 URL

### 适合放 URL

以下类型的状态非常适合存放在 URL 中：

* 搜索关键字、筛选条件
* 分页、排序
* 视图模式（列表/网格、主题）
* 日期范围、时间段
* 当前选中项或激活 Tab
* 影响内容展示的 UI 配置
* 功能开关或 A/B 试验标签

### 不适合放 URL

以下状态不应放进 URL：

* **敏感信息**（密码、令牌、个人隐私）
* 临时 UI 状态（弹窗、展开状态）
* 正在编辑的表单输入
* 大规模复杂状态（嵌套 JSON 等）
* 高频变化的瞬态状态（光标位置等）

一种简单判断规则是：**如果别人点击这个 URL 之后也应该看到同样的状态，那它应该属于 URL；否则，就不应该放进去。**

---

## 如何在前端实现 URL 状态更新

### 原生 JavaScript

利用现代的 `URLSearchParams` 和 History API 即可：

```js
// 读取查询参数
const params = new URLSearchParams(window.location.search);
const view = params.get('view') || 'grid';

// 更新 URL 参数
params.set('sort', 'date');
const newUrl = `${window.location.pathname}?${params.toString()}`;
window.history.pushState({}, '', newUrl);
```

监听 `popstate` 还可以自动在浏览器前进/后退时恢复状态。

---

### React / 框架 Hooks 示例

例如在 React Router 或 Next.js：

```js
import { useSearchParams } from 'next/navigation';

function Page () {
  const [searchParams, setSearchParams] = useSearchParams();
  const color = searchParams.get('color') || 'all';

  const handleColor = (val) => {
    setSearchParams(prev => {
      const params = new URLSearchParams(prev);
      params.set('color', val);
      return params;
    });
  };
}
```

这种方式将 URL 与组件状态紧密绑定。

---

## URL 状态管理的最佳实践

### 避免默认值污染 URL

不要把默认值写入 URL：

```txt
// 差的做法
?page=1&sort=default

// 好的做法
// 只写改变过默认值的部分
```

默认值最好由代码处理，而不是塞进链接。

---

### 输入节流与历史记录

对于搜索输入等高频变更，使用节流与 `replaceState` 以避免历史记录泛滥：

```js
import debounce from 'lodash/debounce';

const updateSearch = debounce((q) => {
  params.set('q', q);
  history.replaceState({}, '', '?' + params.toString());
}, 300);
```

`replaceState` 不会生成新的历史记录条目，对于微调非常合适。

---

### pushState vs replaceState

* **pushState**：用于真正意义的导航，例如改变主要过滤条件或分页
* **replaceState**：用于细微 UI 调整，例如搜索框输入、局部更新

---

## URL 是一种契约

精心设计的 URL 不只是状态容器，它还是一种 **契约**：对用户、开发者以及机器表明状态边界与语义。

一个好的 URL 能清晰表达意图，让人一眼就能看懂。例如：

```
https://example.com/products/laptop?color=silver&sort=price
```

相比模糊的参数键值，这样的 URL 能更好沟通内容含义。

---

## 常见反模式

以下是一些常见的错误用法：

* 把所有状态都放在内存中，刷新即丢失
* 在 URL 中泄露敏感数据
* 参数命名不一致或不具语义
* 把大规模复杂状态编码如 Base64 JSON 丢进 URL
* 错误使用 `replaceState` 导致浏览器后退失效

---

## 结语

PrismJS 的 URL 给了我们一个启发：**好的 URL 不仅指向内容，更描述了用户与应用之间的对话。它捕获了意图、保留了上下文，并实现了状态共享，这是其他状态管理方案无法完全替代的。**

我们构建了各种状态库（Redux、MobX、Zustand 等），它们各有用途。但有时候，最简洁、最有意义的解决方案就是我们一直都有的 —— `URL`。
