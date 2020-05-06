# [日积跬步]系列之2020年



## 20200506

**Java中的 int == Integer 比较**

今天遇到一个bug，在一个 `Java` 服务中，查询数据库账号，某个字段类型是 `Integer`，然后会判断这个值是否和某个 `int` 相等，结果代码抛异常了。仔细查看对应的数据库字段，发现实际字段值是 `NULL`。

原来，当 `==` 运算符是两侧是 `int` 和 `Integer` 时，java会自动对 `Integer` 进行*解包*，调用 `intValue()`方法。这时候，如果 `Integer`对象是 `null`的，那么就会抛出 `NullPointerException`。

参考下面代码

```java
int a = 123;
Integer b = null;  // 这里比较直观，但是如果 b 是一个从数据库查询出来的字段，就不太容易发现问题了
if (a == b) {  // 这里会抛 NullPointerException 异常，因为会隐式调用 b.intValue() 方法
  System.out.println("wont print");			
}
```

看上去，`int == Integer` 这种方式，最好还是改成 `Integer.equals(Integer)` 这种比较好一点。

参考 [java 文档 Equality Operators](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.21.1)



## 20200413

**TypeScript中的 void 类型**

今天看一篇关于 `TypeScript` 的文章，里面有下面的一段代码

```typescript
type WithToString = {
    toString: () => void;
};
const a: WithToString = 111;
console.log(a.toString());
```

这里有个疑问，`Object.prototype.toString` 的类型定义是 `() => string` ，为什么能够赋值给 `() => void` 类型的变量呢？

为了验证，写了如下demo来测试

```typescript
type T2 = () => void;
// 下面赋值没问题，为啥呢？
const foo: T2 = function(){ return 'aaa';};
const out = foo();
out.length; // 这里会报错，因为 out 是 void 类型的，而不是string
```

官网文档里没有明确这个，但是在几个issue里，找到一些蛛丝马迹。

大概意思是，我们都知道，TS里，**子类型的值可以赋值给父类型的变量** 。在这里，我们可以认为 `void` 类型比具体的一些类型，比如 `number` `string` 之类的，更宽泛，因此可以接受这些具体类型。

虽然可以赋值，但是执行函数后返回值的类型，却是 `void` 的，并**不是** 实际的真实类型。参考上面的 `out` 变量，它并不是 `string` 类型的，不能访问任何string类型的属性。

其实，`void` 类型的用法，主要是用在我们**不希望**调用者关心函数返回值的情况下，比如通常的**异步回调函数**，这种情况下返回值是没有意义的。

参考文档

* [documentation Clarify the semantics of void](https://github.com/microsoft/TypeScript/issues/20006) 
* [Function of return type void in interface can return anything](https://github.com/Microsoft/TypeScript/issues/9603) 

