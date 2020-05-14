



本文主要是阅读dart官方文档 [dart核心库一撇](https://dart.dev/guides/libraries/library-tour) ，记录一些知识点。

dart核心库包含了以下这些

* [dart:core](https://dart.dev/guides/libraries/library-tour#dartcore---numbers-collections-strings-and-more)  最基础的库，dart自动导入的。包括各种内置数据类型，集合，以及语言核心
* [dart:async](https://dart.dev/guides/libraries/library-tour#dartasync---asynchronous-programming)  异步编程相关，`Future`  以及 `Stream` 
* [dart:math](https://dart.dev/guides/libraries/library-tour#dartmath---math-and-random)  数学相关常量和函数
* [dart:convert](https://dart.dev/guides/libraries/library-tour#dartconvert---decoding-and-encoding-json-utf-8-and-more)  编码、解码 相关，包括 `JSON`  `UTF-8` 等
* [dart:html](https://dart.dev/guides/libraries/library-tour#darthtml)   这个是给浏览器里使用的，包括 `DOM` 操作等
* [dart:io](https://dart.dev/guides/libraries/library-tour#dartio)  提供 IO操作，包括 Dart VM，flutter，server端 以及 命令行脚本



## dart:core 

数字、字符串、集合等。

提供全局的 `print` 函数，打印任何对象到控制台(调用对象的 `toString()` 方法)。

### 数字

提供string和num互相转换的方法，包括 

* string转换num：`num.parse('15')`，`int.parse('0x42')`，`double.parse('3.14')`
* string转int可以指定几进制，比如 `int.parse('42', radix: 16) == 66` 
* num转string：`num.toString()`  `double.toStringAsFixed(2)` `double.toStringAsPrecision(2)`

### 字符串和正则表达式

在dart里，`String`是 **不可变** 数据类型，这意味着，在 `String` 上的所有修改操作，都不会修改原 `String`，而是返回新的修改之后的 `String`。

和JS一样，`String` 也有 `length`属性，获取字符串长度。

`String` 还有一些常用方法，比如 `split()`, `contains()` `startsWith()`  `endsWith()` `indexOf()` `substring()` `toUpperCase()`  `toLowerCase()`  `trim()` 等。还可以通过 `isEmpty` 属性来判断是否长度为0，还有对应相反含义的 `isNotEmpty`属性。`replaceAll()`可以配合 `RegExp`来对字符串内部子串进行全部替换。

不同于JS，dart里有一个 `StringBuffer`，用来动态的创造 `String`，用法如下

```dart
var sb = StringBuffer();
sb
  ..write('Use a StringBuffer for ')
  ..writeAll(['efficient', 'string', 'creation'], ' ')
  ..write('.');

var fullString = sb.toString();
```

dart里的正则表达式是 `RegExp` 类。定义一个正则，比如 `var numbers = RegExp(r'\d+');` 注意，在 `\d+`前面，有一个 `r`，这代表 *原始字符串*，不会识别后面字符串里的转义字符，实际查找的正则就是 `\d+`，代表查找1个或多个数字；如果去掉前面的 `r`，那么经过字符串转义处理之后，实际查找的正则是 `d+`，代表查找1个或多个字母d。

可以用 `String.contains(RegExp)` 来检测某个字符串内是否包含符合正则的子串，也可以换成对应的 `RegExp.hasMatch(String)` ，一个意思，只是把主角换成了正则。大概用法如下:

```dart
var numbers = RegExp(r'\d+');
var someDigits = 'llamas live 15 to 20 years';

// Check whether the reg exp has a match in a string.
assert(numbers.hasMatch(someDigits));

// Loop through all matches.
for (var match in numbers.allMatches(someDigits)) {
  print(match.group(0)); // 15, then 20
}
```

### 集合类

dart核心库提供了一系列的集合相关类，比如 `list`  `set`  `map`。

`list`类似JS的数组，也提供了一些增删查改的方法，看下大概用法吧

```dart
// Use a List constructor.
var vegetables = List();

// Or simply use a list literal.
var fruits = ['apples', 'oranges'];

// Add to a list.
fruits.add('kiwis');

// Add multiple items to a list.
fruits.addAll(['grapes', 'bananas']);

// Get the list length.
assert(fruits.length == 5);

// Remove a single item.
var appleIndex = fruits.indexOf('apples');
fruits.removeAt(appleIndex);
assert(fruits.length == 4);

// Remove all elements from a list.
fruits.clear();
assert(fruits.isEmpty);

// Sort a list.
fruits.sort((a, b) => a.compareTo(b));

// This list should contain only strings.
var fruits2 = List<String>();
```

dart里，`set`是一堆元素的**无序**集合，集合里每个元素都是不重复的；因为是无序的，每个元素没有对应集合里的位置 `index`，所以就不能通过index来获取某个元素。

```dart
var ingredients = Set();
ingredients.addAll(['gold', 'titanium', 'xenon']);
assert(ingredients.length == 3);

// 由于集合里已经存在 gold 这个string了，再次添加，集合不会新增元素
ingredients.add('gold');
assert(ingredients.length == 3);

// 从集合中删除 gold 这个元素
ingredients.remove('gold');
assert(ingredients.length == 2);

// 查找集合是否包含某个元素
assert(ingredients.contains('titanium'));

// 检查是否全部元素都在集合里
assert(ingredients.containsAll(['titanium', 'xenon']));
```

`map`，通常又称作 `字典` 或者 `哈希对象`，是一系列**无序**键值对的集合。

和JS里 **不一样** 的地方是，在dart里，`object` 不是 `map`。其实JS新的规范里，也新增了 `Map` 类，和dart里的应该是能对应上的。

大概用法如下，也是一些初始化、查找key是否存在、删除key 之类的

```dart
// Maps often use strings as keys.
var hawaiianBeaches = {
  'Oahu': ['Waikiki', 'Kailua', 'Waimanalo'],
  'Big Island': ['Wailea Bay', 'Pololu Beach'],
  'Kauai': ['Hanalei', 'Poipu']
};

// Maps can be built from a constructor.
var searchTerms = Map();

// Maps are parameterized types; you can specify what
// types the key and value should be.
var nobleGases = Map<int, String>();

// 检查map里是否含有某个key
assert(nobleGases.containsKey(54));

// 删除map里某个key以及对应的value
nobleGases.remove(54);

var hawaiianBeaches = {
  'Oahu': ['Waikiki', 'Kailua', 'Waimanalo'],
  'Big Island': ['Wailea Bay', 'Pololu Beach'],
  'Kauai': ['Hanalei', 'Poipu']
};

// 获取map里全部的key，返回一个 Iterable 对象，包含 length 属性
var keys = hawaiianBeaches.keys;

assert(keys.length == 3);
```

`list`  `set`  和 `map` 都有一些同样的属性或方法，大概如下

* `isEmpty`  `isNotEmpty` 检查集合是否空的，是否非空
* `forEach()` 方法，遍历集合里每个元素

有个类 `Iterable` 需要关注下，`list` 和 `set` 都是 `Iterable` ，`map`不是，但是 `map` 的 `keys` 返回的键集合是 `Iterable`。

### Uri类

`Uri`类提供一系列和 `URL`打交道的编码、解码方法。

* `Uri.encodeFull` 对完整的URL进行编码，会保留URL中特殊含义的字符，比如 `/:&#`
* `Uri.encodeComponent`  对字符串中在URL里有特殊含义的字符进行编码
* `Uri.parse`  对一个URL字符串解析成 `Uri` 对象

具体可以查看  [Uri API reference](https://api.dart.dev/stable/dart-core/Uri-class.html) 

### 日期和时间

dart里提供 `DateTime` 这个类来操作日期。

根据下面示例代码，`DateTime` 里指定的 月份、月份里的天，都是从 **1** 开始的，而不是 **0** 开始计算。

```dart
// 获取当前时间
var now = DateTime.now();

// 使用本地时区
var y2k = DateTime(2000); // January 1, 2000
// 指定时间为 UTC 时间
y2k = DateTime.utc(2000); // 1/1/2000, UTC

// **注意**，这里代表 2020年1月2日 
y2k = DateTime(2000, 1, 2); // January 2, 2000

// 从毫秒时间戳获取时间
y2k = DateTime.fromMillisecondsSinceEpoch(946684800000,
    isUtc: true);

// 从字符串解析日期
y2k = DateTime.parse('2000-01-01T00:00:00Z');

// 1/1/2000, UTC
y2k = DateTime.utc(2000);
// 使用 millisecondsSinceEpoch 属性来获取毫秒的时间戳
assert(y2k.millisecondsSinceEpoch == 946684800000);
```

日期的修改，日期之前的差值，都可以通过 `Duration` 这个类来计算。



## dart:async 异步编程

dart提供了两个和异步编程相关的类 `Future` 和 `Stream` 。

`Future` 就是JS里的 `Promise`，换了身马甲而已，也是 `then()` 方法来注册异步回调拿结果，使用 `catchError()` 来捕获异常(也是换了个名字)。也提供了并发等待多个异步回调结果的 `Future.wait([])` ，和JS里的 `Promise.all()` 看上去没有任何区别。

同样，在JS里，有了 `async`  `await`  之后，就 **不推荐** 使用 `Promise.then` 方法来异步编程了；在 dart里，也推荐使用 `async`  `await`  来异步编程，不直接使用 `Future.then` 。

dart里的 `Stream` ，一方面和 `node.js`  里的 `stream` 类似，比如读取文件流；另一方面，在 `DOM` 操作上，对应的绑定事件也是流，可以认为DOM会异步的触发这些事件。比如

```dart
// 绑定某个元素的点击事件, onClick 属性就是一个 Stream
querySelector('#submitInfo').onClick.listen((e) {
  // When the button is clicked, it runs this code.
  submitData();
});

// 读取文件流中，Stream 的使用
Future readFileAwaitFor() async {
  var config = File('config.txt');
  Stream<List<int>> inputStream = config.openRead();

  var lines = inputStream
      .transform(utf8.decoder)
      .transform(LineSplitter());
  try {
    await for (var line in lines) {
      print('Got ${line.length} characters from stream');
    }
    print('file is now closed');
  } catch (e) {
    print(e);
  }
}
```



## dart:math 数学以及随机数

`dart:math` 库提供一些列的数学相关的函数以及常量，比如 `sin()` `cos()`  `max()` `min()` 函数，还有常量 `pi`  `e`  `sqrt2`  。不太理解的是，为什么这些函数以及常量都是 **全局** 的，窃以为像JS里都放在 `Math` 类下更优雅一点吧。

可以通过 `Random` 类来产生随机数、随机bool值。像下面这样

```dart
var random = Random();
random.nextDouble(); // 在[0, 1)之间随机生成浮点数(注意是 **左闭右开** 区间)
random.nextInt(10); // 在 [0, 9] 之间随机生成int
random.nextBool(); // 随机产生 true 或者 false
```



## dart:convert 编码、解码相关

`dart:convert` 库包含了对 `JSON` `UTF-8` 等的编码、解码的支持。

使用 `jsonDecode()` 将一个JSON字符串解码为dart的对象，可能是 `list`  `map`  等。

使用 `jsonEncode()` 将dart的对象转化成JSON字符串。注意，只有这些类型可以直接转换成JSON字符串： `int`  `double`  `String`  `bool`  `null`  `List`  `Map`(key是string类型的时候)。对于其他dart对象，调用 `jsonEncode` 转换JSON字符串时，可以提供额外的一个 `encode()` 函数作为第二个参数，这个函数返回上面几种可以直接转换的类型之一；如果没有提供 `encode` 作为第二个参数，那么系统会调用对象的 `toJson()` 方法（这个和JS里是类似的吧）。

别的针对 `UTF-8` 的操作，包括 `utf8.decode()`  `utf8.encode()`  ，先跳过吧。



## dart:html 开发web应用

`dart:html` 库是开发 **web** 应用的，先跳过吧，毕竟看dart，也不是为了开发webapp ……



## dart:io IO操作

`dart:io` 库提供一些列的API，用来操作 文件、目录、进程、sockets、WebSockets、以及http客户端和服务器。

库提供的大部分API都是异步的，和 `Node.js` 一样，针对个别同步接口，会在对应函数名加上 `Sync` 后缀来标示。

读取文件内容有2种方式：一次性全部读取、通过 `Stream` 来读取。如果要一次性全部读取，必须保证有足够的内存能够存储文件内容。

通过 `File` `Directory` 这两个类分别操作 文件和目录。

dart还可以搭建server端，通过 `HttpServer`  `HttpRequest` 类，可以启动server端，处理http请求。

如果是要主动发送http请求，就要用到 `HttpClient` 类。



## 最后

大概过完了，继续看后面还有啥吧







