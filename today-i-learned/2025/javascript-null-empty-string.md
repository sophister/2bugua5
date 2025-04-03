# Javascript: null == ""，true 还是 false



## 问题

最近遇到一个生产问题，原始代码如下：

```javascript
let levelCode = "";
if (this.$store.state.levelCode4 == "") {
  levelCode = this.$store.state.levelCode3;
} else {
  levelCode = this.$store.state.levelCode4;
}
```

其实看前辈的代码，还是基本能猜到他的意图，`levelCode` 字段优先取 `levelCode4` ，如果没有值，那么就取 `levelCode3`。看这个意思，有可能存在 `levelCode4` 为空的情况。但根据线上反馈，有用户请求里并没有带上 `levelCode` ，通过该账号的信息，我们发现对方的数据是这样的：

```json
{
    levelCode4: null,
    levelCode3: "xxxx"
}
```

看这个数据，我想不应该啊，走到 `if` 分支的时候，应该就能判断出来，直接取 `levelCode3` 了，为什么会有问题呢？毕竟 `null == ""` 这个应该是 `true` ，没有什么疑问。

眼睛看不出问题，那就只能上断点了……

原来，`null == ""` 这个表达式的值是 `false` ！！

把上面代码发到一个群里，好像大部分同志也没能一眼看出这个问题，还好健忘的不止我一个🤣

还是问问ChatGPT吧（sorry 谷哥，你做得很好，但你已经不适合我了🙄），给出的答案是正确的，但是部分知识点说的不够完整。

> 在 JavaScript 中，表达式 `null == ""` 的结果是 `false`。这是因为在使用宽松相等运算符（`==`）时，JavaScript 会进行类型转换（Type Coercion）以比较不同类型的值。以下是该表达式求值过程的详细步骤，包括涉及的类型转换，并附上相关的 MDN 文档链接：
>
> 1. **理解操作数的类型和值**：
>    - 左操作数：`null`，表示“无值”或“空值”。 citeturn0search2
>    - 右操作数：`""`，一个空字符串，类型为字符串。
> 2. **应用宽松相等运算符的规则**： 根据 JavaScript 的宽松相等比较规则，当比较 `null` 或 `undefined` 与其他值时，只在与 `null` 或 `undefined` 互相比较时才会返回 `true`。否则，`null` 和 `undefined` 只与自身相等，不会与其他类型的值相等。 
> 3. **具体比较过程**：
>    - `null` 仅与 `undefined` 相等，即 `null == undefined` 为 `true`。
>    - `null` 与任何其他值（如字符串、数字、对象等）比较时，都会返回 `false`。
>
> 因此，`null == ""` 的结果是 `false`，因为 `null` 只与 `undefined` 相等，而不与空字符串相等。
>
> **注意**：在进行相等比较时，建议使用严格相等运算符（`===`），因为它不会进行类型转换，要求值和类型都相同。例如，`null === ""` 也会返回 `false`，因为它们的类型不同



根据MDN文档，也明确提到了 `null` 或者 `undefined` 在参与 `==` 比较的规则：

> If one of the operands is `null` or `undefined`, the other must also be `null` or `undefined` to return `true`. Otherwise return `false`.

意思也很明显，当有 `null` 或者 `undefined` 参与 `==` 比较时，只有下面这些表达式才为 `true`

```javascript
null == null // true
null == undefined // true
undefined == undefined // true
undefined == null // true
```



## 怎么写

其实回到上面这个bug，通常我们不会这样去判断，一般有下面2种方式吧

1. 使用 非`!` 来判断是否为空  `if (!this.$store.state.levelCode4) {levelCode = levelCode3}`
2. 使用三元表达式 `levelCode = this.$store.state.levelCode4 || levelCode3`

上面的判断 ，会考虑 `levelCode4` 是 `null/undefined/空字符串/false/0` 等情况下，都会使用 `levelCode3` 来赋值。



如果我们只希望在 `levelCode4` 是 `null/undefined` 这种情况下才用 `levelCode3` ，那可以用新一些的JavaScript语法——空值合并运算符??：

`const levelCode = this.$store.state.levelCode4 ?? levelCode3;`



* [MDN等于判断（==）](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Equality)
* [MDN空值合并运算符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)