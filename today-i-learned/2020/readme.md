# [日积跬步]系列之2020年



## 20200518

**macOS Catalina 10.15.4系统下git pull/push 卡住的问题**

上周不小心升级到了最新的 `macOS Catalina 10.15.4` 版本，升级之后，发现 `git` 相关命令(`git pull`  `git push`等)经常卡住没有反应，重启系统之后又恢复正常，但是过几天又异常……

但是也只是访问公司的 `gitlab` 会有异常，访问 `github` 完全没问题……

问了周围同事，都是正常的……

后来发现，他们都是使用的 `http` 协议 `clone` 代码，而我是走的 `ssh` 协议。

再对比公司的 `gitlab` 和 外部的`github` 服务，我还是都用的 `ssh` 协议，但是服务的端口不一样！公司的`gitlab` ssh协议的端口不是正常的 `22` 端口，而是一部比较大的值(大于`8192`) ，比如是 `23456` 。

Google之后，终于发现 [这篇stackoverflow](https://apple.stackexchange.com/questions/386821/git-hangs-with-a-ssh-remote-uri-after-10-15-4-update) 和我遇到情况完全一样，升级系统之后，`git` 相关命令无法使用。

根据上面的 stackoverflow 步骤，应该是 macOS 自带的 `/usr/bin/ssh` 有问题，按照如下步骤，安装新的 `ssh` 客户端

```shell
brew install openssh
```

安装过程中，可能会遇到如下报错信息

```shell
Error: The `brew link` step did not complete successfully
The formula built, but is not symlinked into /usr/local
Could not symlink sbin/sshd
/usr/local/sbin is not writable.

You can try again using:
  brew link openssh
```

可能是 `/usr/local/sbin` 目录不存在，需要手动创建，并且赋予当前用户权限

```shell
# 创建目录，如果不存在的话
sudo mkdir /usr/local/sbin
# 修改目录的所有者
sudo chown -R $(whoami) /usr/local/sbin
# 重新link
brew link openssh
```

最后，记得确保 `/usr/local/bin` 在你的 `PATH` 环境变量靠前的位置。

记录下 macOS 的 ssh 和 brew 安装的 openssh 版本信息的差异吧

```shell
jess@jessedeMacBook-Pro ~ % /usr/bin/ssh -V
OpenSSH_8.1p1, LibreSSL 2.7.3
jess@jessedeMacBook-Pro ~ % /usr/local/bin/ssh -V
OpenSSH_8.2p1, OpenSSL 1.1.1g  21 Apr 2020
```



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

