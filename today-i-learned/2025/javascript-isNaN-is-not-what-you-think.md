# JavaScript世界的卧龙凤雏：NaN和isNaN

作为一个搬砖经验10年+的老页面仔，我知道 `NaN` 很牛逼，但是我没意识到，`isNaN()` 这个全局函数也很牛逼。

 这次要分享的，是我在项目里遇到的一个**“小坑”**，看似很简单的一个 `isEmpty` 判断，却被一个老朋友——`isNaN()` 给坑惨了。

 这事儿让我对 JavaScript 里 `isNaN()` 的行为有了全新的认识，也认识了全新的朋友——`Number.isNaN()`。

 这个bug深刻的揭示了一句歌词：

 >结识新朋友，不忘老朋友

 我们整天疲于学习使用各种新技术、API、特性，有时却容易在最不起眼的地方栽个大跟头。

------

## 事情的起因

当时我在做一个表单校验功能，需要判断一个值是不是“空的”。于是我很自然地写了这么一个函数：

```ts
function isEmpty(value: any): boolean {
  return value === undefined || value === null || value === '' || isNaN(value);
}
```

思路很简单：

- `undefined` / `null`：空
- 空字符串 `''`：空
- `NaN`：空（用 `isNaN` 检测）

结果上线后，测试同事跟我说：“为啥我输入 `'abc'`（字符串）的时候，系统提示这个值是空的？”
 我当时一脸懵：`'abc'` 怎么会被当成空值呢？

------

## 问题复现

我在控制台随手一敲：

```js
isNaN('abc'); // true
```

好家伙，这下我明白了。
 原来，**全局 `isNaN` 根本不只是判断值是不是 `NaN`**，它还会做一些你没预料到的事情。

------

## 认识一下 JS 里的两种 “isNaN”

其实，JavaScript 里有两种相关的 `isNaN`：

### 1. 全局 `isNaN(value)`

这个历史悠久的家伙，在判断前会**先把参数转成 Number 类型**，再检查结果是不是 `NaN`。
（具体参考MDN文档：[isNaN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/isNaN)）

 所以才会有这些神奇的结果：

```js
isNaN('abc');      // true   -> 'abc' 转数字是 NaN
isNaN('123');      // false  -> '123' 转数字是 123
isNaN(undefined);  // true   -> undefined 转数字是 NaN
isNaN('');         // false  -> '' 转数字是 0
isNaN('   ');      // false  -> 空格字符串转数字是 0
```

是不是很“热心”地帮你做了类型转换？问题就在这。

### 2. `Number.isNaN(value)`

这是 ES6 新增的，**不会做类型转换**，只有当值的类型是 `number` 且本身就是 `NaN` 时才返回 `true`：

```js
Number.isNaN(NaN);      // true
Number.isNaN('abc');    // false
Number.isNaN(undefined);// false
```

一句话总结：

> - **全局 `isNaN`**：先转成数字再判断 → 容易误伤
> - **`Number.isNaN`**：严格判断，只认真正的数值 NaN

------

## 为什么我会翻车？

回到我的 `isEmpty`：

```ts
function isEmpty(value: any): boolean {
  return value === undefined || value === null || value === '' || isNaN(value);
}
```

因为用了全局 `isNaN`，当 `value` 是 `'abc'`、`'NaN'`（字符串）、甚至是 `undefined` 时，都可能被判成空值。
 结果就是，我本来想判断“值是不是 `NaN`”，却变成了“值是不是不能被转成数字”。

这显然不是我想要的。

------

## 我的修复方案

既然问题出在全局 `isNaN` 的隐式转换，那就两步走：

1. 按类型来判断（数字就是数字，字符串就是字符串）
2. 对数字用 `Number.isNaN`，对字符串做 `trim()` 检查空白

改造后的代码是这样的：

```ts
function isEmpty(value: any): boolean {
  if (value === undefined || value === null) return true;

  if (typeof value === 'string') {
    return value.trim() === '';
  }

  if (typeof value === 'number') {
    return Number.isNaN(value);
  }

  // 其他类型暂时不考虑
  return false;
}
```

------

## 彩蛋：NaN 还有个神奇特性

作为合格的web前端页面仔，大家应该都知道下面这个宇宙无敌的特性吧：
**NaN 是 JavaScript 里唯一一个不等于自身的值。**

```js
NaN === NaN; // false
NaN !== NaN; // true
```

这就意味着，你也可以用这种方式来判断 NaN：

```js
function isReallyNaN(value) {
  return value !== value;
}

isReallyNaN(NaN);    // true
isReallyNaN('abc');  // false
isReallyNaN(123);    // false
```

这种方法简短高效，但可读性堪忧。第一次看到的人可能会以为你在写谜语（**刚才cursor就提示我，这段代码是无用的**）。😂

我个人建议在生产代码里用 `Number.isNaN`，而 `value !== value` 就留作分享时的趣味知识吧。

## 额外感悟

写工具函数时，我以前总觉得直接用内置 API 就行，没想到一个老函数还能暗藏陷阱。
 现在我更倾向于**明确类型 → 针对性判断**，这样既安全又符合业务语义。

另外，如果你要兼容很老的浏览器，可以加个 polyfill：

```js
Number.isNaN = Number.isNaN || function(v) {
  return typeof v === 'number' && isNaN(v);
};
```

------

## 结尾

这次的坑虽然不大，但提醒了我一个道理：
 **不要过度依赖“看似简单”的内置方法，它们的历史包袱可能会让你吃亏。**

如果你也有过类似的“老 API 坑人”经历，欢迎在评论区分享，让我们一起少踩点坑。

