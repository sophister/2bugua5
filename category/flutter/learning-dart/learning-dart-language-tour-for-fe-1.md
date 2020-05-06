#前端学dart(1): 从未入门到快放弃



本文基本都是对应dart官方的 [dart 语言入门之旅](https://dart.dev/guides/language/language-tour) 来写的，简单记载下看的过程中一些心得(&吐槽)。



## final 和 const



### JavaScript

`JavaScript` 新的ES标准加上了 `const` 修饰符，但还没有 `final` 修饰符(如果没记错的话，ES发展太快了……)。

在JS里，如果一个变量 **指向** 一个值之后，不会再修改变量指向的值，那就应该用 `const` 来定义变量。

但是，如果 `const` 变量是指向的一个 **对象类型** ，比如一个JSON字面量，那么这个JSON字面量里的字段，是可以被修改的。

### Dart

 dart里不仅有 `const` ，还有 `final`(不知道搞两个干啥，Google之后发现有下面这些不同点)。

* `final` 是修饰**变量**的，`const`是修饰**值**的。`final`的变量不能重新赋值，但是变量引用的对象，可以修改内部的字段值；`const`的变量，变量本身以及变量引用对象内部的字段，都不能重新赋值
* `const`的值，必须在 **编译期** 能够确定下来；凡是在 **编译期** 不能确定的，都不能用 `const`，只能用 `final`
* 类上面的常量**实例属性**，只能用 `final`，不能用 `const` 
* 类的**常量属性**，应该用 `static const` 修饰



参考 [https://stackoverflow.com/questions/50431055/what-is-the-difference-between-the-const-and-final-keywords-in-dart](https://stackoverflow.com/questions/50431055/what-is-the-difference-between-the-const-and-final-keywords-in-dart) 



## 数据类型



### JavaScript

JS里包含这些类型：`number`   `string`  `boolean`  `undefined`  `null`  `symbol`   `bigint` ，以及 `object` 引用类型。



### Dart

dart的数据类型更加细

* `num` 数字类型，是 `int` `double` 的父类

* `int`  如果定义数字时，初始值不带小数点，会默认为 int 类型

* `double`  如果定义数字初始值包含小数点，默认为 double 类型。当然也可以明确声明变量类型，像这样 `double z = 1` 

* `string` 可以用单引号或者双引号来定义。也有字符串里插值，格式和JS很像，也支持 `"a ${variable}"`，甚至还可以把大括号去掉。可以定义多行的字符串，通过 三个双引号或者单引号 来定义，比如这样

  * ```dart
    // 定义多行字符串
    var str = ```
      first line
      second line   end
      third
    ​```
    ```

  * 支持 *raw string*，以 `r` 开头，这样不会解析字符串的转义字符，比如 `var s = r'In a raw string, not even \n gets special treatment.';` 

* `bool` 布尔类型，没啥可说的。有个和JS里的不同点是，dart是强类型，需要bool的地方，必须是传一个bool值，不能传别的类型。比如 `if(condition){}` 里，`condition`只能是bool类型的值；但是在JS里，`condition` 可以是任何值

* `lists` 列表(有序对象数组)，相当于JS里的 `Array` 。

* `set` 无序集合，集合里的元素是惟一的。定义集合字面量方法 `var halogens = {'fluorine', 'chlorine', 'bromine', 'iodine', 'astatine'};` 如果是定义空集合，那**必须**要带上集合类型，比如 `var names = <String>{};` 或者 `Set<String> names = {};` 。如果没有集合类型，那定义的就不是 `set`，而是 `map` ，像这个 `var names = {};`  是定义了一个 `map`  

* `map` ，键值对集合的对象，这个类似JS里的 `JSON` 了。如果访问的key在map里不存在，那么返回 `null` ，这个和JS也**不一样**。

* `runes` 处理 unicode 编码相关的字符串操作

* `symbol` 暂时不知道干啥的，感觉和JS里的symbol不一样。



## 函数 Function



`function` 函数在dart里也是对象，一等公民，这个和JS一样，可以赋值给变量，或者作为函数参数来传递。

* 定义函数，不需要 `function` 关键字，但是**建议**要写上参数类型和返回值类型。
* 如果函数体只包含一个**表达式**，那也可以使用 **箭头函数**。但是这个显然和JS里的箭头函数**不一样**，JS里箭头函数可以包含任意行语句，并且会绑定上下文this。
* 函数参数可以有必需参数和可选参数。如果是可选参数，可以有2种方式来定义，只能选择其中一种。比如 `void enableFlags({bool bold, bool hidden}) {...}` 或者 `String say(String from, String msg, [String device]) {...}` 。如果是通过 `{bool bold, @required bool hidden}`这种方式定义可选参数，那可以通过 `@required` 注解来设置某个参数是必须的！
* 可选参数可以有默认值，`void enableFlags({bool bold = false, bool hidden = false}) {...}` ，默认值必须是 **编译期常量** 。如果不提供默认值，那可选参数的默认值是 `null` 
* 特殊的 `main()` 函数，每个dart程序都必须有一个定义在顶级作用域的main函数，作为程序入口
* 支持 **匿名函数**，这个和JS一样，没啥可说
* 所有dart函数都有返回值，如果没有明确指定返回值，那么默认返回 `null` 



## 操作符 Operators



暂时没啥可说的，详细表格看 [官方文档](https://dart.dev/guides/language/language-tour#operators)吧，不过dart里是有 **操作符重载** 的，记忆里好像上次看到这个词，还是在大学学 *C++* 的时候

### 数学运算

比JS多了一个 `~/` 运算符，两个数字相除且返回整数部分。看这个例子

```dart
assert(5 / 2 == 2.5); // Result is a double
assert(5 ~/ 2 == 2); // Result is an int
```

### 比较运算符

`==`相等操作符，和JS里可以说完全不一样了！对于 `x == y`的判定，具体规则如下：

* 如果x或者y之一是 `null`：如果都是`null`，结果是 `true`；否则结果是 `false`
* 返回方法调用的值 `x.==(y)`。在dart里，`==`操作符是调用对应运算符左值的方法

### 类型检测

可以使用 `as`  `is`  `is!` 在 **运行时** 进行类型相关的检查和转换。

*  `as` 强制类型转换。比如 ` foo as Persion`，如果 foo 是 `null` 或者 不是 `Persion` 实例，会在运行时抛异常！
* `is` 判断类型。`obj is T` 返回 `true` 如果 `obj` 实现了类型`T` 所定义的接口

### 赋值操作符

比起JS，dart里多了一种 ` b ??= value` 的运算符，只有当 b 是 `null` 时，才会执行赋值。

```dart
// Assign value to a
a = value;
// Assign value to b if b is null; otherwise, b stays the same
b ??= value;
```

###  条件表达式

dart比起JS，除了传统的 `condition ? expr1 : expr2` 三元表达式，还多了一种 `expr1 ?? expr2` 表达式：如果 expr1 != null，则返回 expr1 ；否则返回 expr2。

### 级联符号

dart有级联符号 `..` (两个点)，方便在同一个对象上进行多次操作。比如

```dart
querySelector('#confirm') // Get an object.
  ..text = 'Confirm' // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));
```

### 其他操作符

有一个 `foo.?prop` 的属性访问操作符，貌似JS新规范也在考虑加上么(好像在哪儿看到过)。就是当 `foo` 不是 `null` 的时候，才会去返回 `foo.prop` ；如果 `foo` 是 `null`，则直接返回 `null`



## 控制流

`if/else` ，这个和JS不同，在dart里，必须要求条件是 `bool` 类型的值！



## 异常(捕获)

在dart里，可以 `throw` 任何对象作为异常；当然，推荐抛出的异常是 `Exception` 或者 `Error` 的子类。

要捕获异常，需要使用 `on` 或者 `catch` 。如果要指定捕获的异常类型，就需要用 `on` ；如果要访问异常对象，就需要用 `catch` ，看下面例子：

```dart
try {
  breedMoreLlamas();
} on OutOfLlamasException {
  // A specific exception
  buyMoreLlamas();
} on Exception catch (e) {
  // Anything else that is an exception
  print('Unknown exception: $e');
} catch (e) {
  // No specified type, handles all
  print('Something really unknown: $e');
}
```

`catch` 可以声明2个参数，第一个是异常对象，第二个是抛出异常的堆栈对象(`StackTrace`类)。

可以在 `catch` 块内重新抛出异常，使用 `rethrow;` 即可，后面不跟任何参数(搞不懂为啥不直接用 `throw`)。

同样，`try` 可以有一个 `finally` 块，如果有匹配的 `catch` 块，`finally` 块会在 `catch` 执行之后才执行；如果没有匹配的 `catch` ，在 `finally` 代码块执行结束之后，异常会继续往上层抛。



## 类(Class)

dart是面向对象语言，所有的类都是 `Object` 类的子类。每一个类都只能有一个父类。

和JS相比，dart里访问对象的属性，增加了 `foo?.attr` 访问符，只有当 `foo` 不是 `null` 的时候，才会执行，如果是 `null`，不会执行。

要实例化一个对象，dart里的 `new`关键字是可选的，要不要都可以。这一点和JS可**不一样**！



dart里有所谓的 [constant constructor](https://dart.dev/guides/language/language-tour#constant-constructors) ，就是下面这样的，调用之后，可以返回 编译期常量(compile-time constant)

```dart
var a = const ImmutablePoint(1, 1);
var b = const ImmutablePoint(1, 1);

assert(identical(a, b)); // They are the same instance!
```

**这一坨暂时没搞清楚，先放下，等后面再细看吧**

### 类实例属性

dart是 **强类型** 语言(和JS不一样)，因此，在类上定义属性时，需要加上属性的类型，比如下面这样

```dart
class Point {
  num x; // Declare instance variable x, initially null.
  num y; // Declare y, initially null.
  num z = 0; // Declare z, initially 0.
}
```

`Point` 类上有 x/y/z 三个实例属性，x/y在定义时没有给初始值，默认是 `null`，因为dart里一切皆对象，所有未初始化的实例属性值都是`null`。

所有**实例属性**都会生成默认的 `getter` 方法，所有 **非final** 的实力属性都会生成 `setter` 方法。都是通过 `obj.prop` 的方式来读写。

如果在定义一个实例属性的时候就给了初始值，那么在实例对象被创造之前这个属性就有初始值了，这个初始值的赋值是在类构造函数和构造函数属性初始化列表之前执行。

### 类构造函数

dart里构造函数提供了一个语法糖，可以直接在构造函数的参数定义里，初始化类的实例属性，像下面这样

```dart
class Point {
  num x, y;

  // 在构造函数函数体执行之前，初始化实例属性 x, y
  Point(this.x, this.y);
}
```

在dart里，如果定义类的时候没有构造函数，那么系统会生成默认的构造函数，没有参数，默认会调用父类的无参数的构造函数。

构造函数**不能**被继承，子类如果没有定义构造函数，那么子类只会有一个系统默认的构造函数，不会继承父类的构造函数。

#### 命名构造函数

dart里，类可以有多个构造函数，像下面这样定义命名构造函数

```dart
class Point {
  num x, y;

  // 构造函数，通过构造函数参数列表初始化实例属性
  Point(this.x, this.y);

  // 命名构造函数
  Point.origin() {
    x = 0;
    y = 0;
  }
}
```

注意，**构造函数不能被继承，这也包括了命名构造函数**

#### 子类构造函数执行顺序

在dart里，`new`一个类的时候，该类的构造函数执行情况如下

1. 执行**子类**对应构造函数的参数列表初始化实例属性(如果有的话)，比如上面的 `Point(this.x, this.y)`
2. 执行**父类**的无参数构造函数
3. 执行**子类**构造函数的函数体

也可以在子类构造函数的函数体之前，手动显式调用父类的某个构造函数，像下面这样(语法看上去有点怪，为啥不搞成java那样的？)

```dart
class Person {
  String firstName;

  Person.fromJson(Map data) {
    print('in Person');
  }
}

class Employee extends Person {
  // Person 类没有默认的构造函数;
  // 必须手动显式的调用 super.fromJson(data).
  Employee.fromJson(Map data) : super.fromJson(data) {
    print('in Employee');
  }
}
```

调用父类构造函数的参数，是在子类函数体执行之前取值，可以是一个表达式，在表达式里可以访问子类的静态方法`static`，但是**不能**访问子类的 `this。`

#### 重定向构造函数(redirecting constructor)

感觉dart那帮大佬很喜欢发明东西啊，这玩意儿看上去就是函数重载啊，搞不懂为啥不做成java那样的，又整个新的语法😪

```dart
class Point {
  num x, y;

  // 基本的构造函数，别的可以调用这个
  Point(this.x, this.y);

  // 这个构造函数值有x可以设置，y默认是0
  Point.alongXAxis(num x) : this(x, 0);
}
```

####  常量构造函数(Constant constructors)

又来个名词，老外也太喜欢发明词汇了

这个东西和**编译期常量**有关系，先放下吧，后面遇到再看，根本记不住。

```dart
class ImmutablePoint {
  static final ImmutablePoint origin =
      const ImmutablePoint(0, 0);

  final num x, y;

  const ImmutablePoint(this.x, this.y);
}
```

#### 工厂构造函数(Factory constructors)

hmm，又来一个名词……

搞了个 `factory`关键字，放在构造函数前面，说明调用这个构造函数来生成对象，不一定每次都是生成新的对象，有可能对象是从某个缓存的池子里直接返回的，也有可能返回的是这个类的子类对象。

注意，虽然在实例化对象的时候，调用工厂构造函数和其他构造函数没有区别，**但是工厂构造函数不能访问this**。

官网这个例子，演示了通过工厂构造函数，从一个缓存map里，查询和保存同名对象的用途

```dart
class Logger {
  final String name;
  bool mute = false;

  // _cache 就是一个对象池子，会缓存 name 相同的对象
  // 这个 _ 下划线开头的，是限制访问控制的，具体还不太清楚
  static final Map<String, Logger> _cache =
      <String, Logger>{};
  // 这就是所谓的工厂构造函数，注意函数里不能访问this
  factory Logger(String name) {
    return _cache.putIfAbsent(
        name, () => Logger._internal(name));
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}
```



### 类方法

类的实例方法，就是能访问当前实例 `this`的方法。

`getter` `setter` 和JS里差不多，也可以自己通过这两个都增加新的实例属性，但是内部是读写别的属性。在 `++`这些操作里，`getter` `setter` 同样适用，只是为了防止额外的副作用，系统只会调用一次 `getter`，然后将获得的值缓存在一个临时变量里。

dart可以定义抽象类，抽象类可以有抽象方法(不实现方法体即可)。抽象类**不能**被实例化，当然也可以在抽象类上增加工厂构造函数，来让抽象类看上去可以被实例化。

```dart
// 下面这个是抽象类，因此不能被实例化
abstract class AbstractContainer {
  // 抽象类可以定义其他的属性、方法

  void updateChildren(); // 抽象方法，没有方法体，留给子类去实现
}
```

#### 隐式接口( Implicit interfaces)

每个类都会隐式的定义一个接口，这个接口包含了类所定义的全部实例成员，以及类实现的全部interface。

```dart
// Person类隐式的定义了一个接口，包含 _name 和 String greet(String)
class Person {
  // 包含在接口里，但只在这个库内部可见
  final _name;

  // 构造函数，不会包含在接口里
  Person(this._name);

  // 实例方法，包含在接口里
  String greet(String who) => 'Hello, $who. I am $_name.';
}

// 别的类可以实现某个类所定义的接口
class Impostor implements Person {
  get _name => '';

  String greet(String who) => 'Hi $who. Do you know who I am?';
}
```

一个类可以实现(`implements`)一个或者多个其他类所定义的接口。

#### 类继承

dart里，还是用 `extends` 来实现类的继承，在子类里，同样适用 `super` 来访问父类的成员。

子类可以覆盖父类的 实例方法、`getter`、`setter` ，可以使用 `@override`注解来明确告诉系统子类是在覆盖父类的成员。

#### 操作符重载

感觉这个和 C++  有点像，可以进行操作符重载(不知道有没有记错……)。比如可以重载 `+` `-` `==` `>` `<` `>>` `|` 等很多操作符，但是**不包括** `!=`，因为表达式 `e1 != e2` 只是 `!(e1 == e2)` 的语法糖。

```dart
class Vector {
  final int x, y;

  Vector(this.x, this.y);
  // 重载 + 运算
  Vector operator +(Vector v) => Vector(x + v.x, y + v.y);
  // 重载 - 运算
  Vector operator -(Vector v) => Vector(x - v.x, y - v.y);
  // 重载 == 运算，省略
}
void main() {
  final v = Vector(2, 3);
  final w = Vector(2, 2);

  assert(v + w == Vector(4, 5));
  assert(v - w == Vector(0, 1));
}

```

如果重载了 `==`运算符，那么必须重载 `Object`类的 `hashCode` getter。

有一个 `noSuchMethod`方法，猜想还是来自 `Object` 类吧，如果你在某个类中重载了，那么当访问该类实例上不存在的属性和方法时，系统会调用类的这个 `noSuchMethod` 方法。暂时放下，等以后再看有没有啥用。

#### 扩展已有类库方法

dart允许我们给已经存在的类库，添加一些自己的方法。

听上去，怎么感觉很久之前，JS里直接给一些系统类patch一些新的方法呢，尤其是那个 `String.prototype.trim`，早前好像都是直接给 `String.prototype`上添加这个方法来搞的。但是后来有说直接在原生对象上扩展方法不太好……

#### 枚举

没啥可说的，也是 `enum`关键字，定义枚举，每个枚举值，都有一个隐式的getter属性 `index`，从0开始。

#### mixins扩展类

dart只支持继承单个父类，因此为了逻辑复用，又搞了这个 `mixins` 来从多处继承逻辑。

要定义一个 `mixin`，继承自 `Object`，不定义构造函数，使用关键字 `mixin`即可

```dart
// 定义一个mixin
mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}
// 别的类可以“继承”这个mixin，使用 with 关键字，后面跟着一个或多个mixin
class Maestro extends Person
    with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

mixin还可以声明哪些类可以 “继承” 它，在定义mixin的时候使用 `on`后面跟上类名即可，感觉老外真是会玩啊。

```dart
mixin MusicalPerformer on Musician {
  // ···
}
```

PS，在 `React`生态里，不是都摈弃掉通过 `mixin`来复用逻辑了么，不知道在dart生态里，这玩意儿还能王者归来么？

#### 类属性和类方法

在类的属性或方法前面，加上 `static` 关键字，就成了类属性、类方法。这个和java一样，就是属于类的，而不是实例对象的，因此类方法里也就**不能**访问 `this`了。

## 泛型

嗯，泛型先放下吧，看完就忘的节奏……

## 类库和可见性

使用 `import` 来导入一个类库。如果是dart内置的库，使用 `dart:`前缀；如果是包管理器的库，可以使用 `package:`前缀；其他的可以使用本地文件路劲。

如果导入的2个库有同名的标识符，那么可以给库一个别名来引用标识符，避免命名冲突

```dart
import 'package:lib1/lib1.dart';
import 'package:lib2/lib2.dart' as lib2;   // 相当于lib2的别名
// 比如上面两个库都定义了 Element 类，需要分清楚具体是使用哪一个

// Uses Element from lib1.
Element element1 = Element();

// Uses Element from lib2.
lib2.Element element2 = lib2.Element();
```

还可以只导入一个文件的某些部分，比如

```dart
// 只导入 foo
import 'package:lib1/lib1.dart' show foo;

// 导入除了foo之外的全部东东
import 'package:lib2/lib2.dart' hide foo;
```

dart提供了动态加载类库，但是只在 `dart2js` 支持，在 `flutter` `Dart VM` `dartdevc` 上都不支持。



## 异步编程

dart里异步类是 `Future` ，类似JS里的 `Promise` 。和JS一样，还是使用 `async` 关键字来定义异步函数，返回 `Future`，使用 `await` 关键字来获取异步 `Future` 的运行结果。同样， `await`关键字只能用在 `async`函数内部。(好像JS已经在支持顶级作用域直接使用 await 了？不知道是不是看错了)

如果要在应用的入口函数 `main` 里面使用 `await`，必须把 `main` 函数也定义成 `async` 的，像下面这样(为啥要把 async 放在括号后面，一定要显得自己与众不同么)

```dart
Future main() async {
  checkVersion();
  print('In main: version is ${await lookUpVersion()}');
}
```

和JS类似，如果一个函数是 `async` 的，那么返回值一定是 `Future` 。如果异步函数不返回有意义的结果，那么可以将函数返回值标记为 `Future<void>` 。

dart异步相关的类还有 `Stream` 以及对应的语法 `await for`，先略过，整太多了这帮大神。

## 函数类型别名 Typedefs 

dart里，可以用 `typedef` 关键字来定义一个函数类型的别名，感觉这个和 `TypeScript` 里的`type`有点像。不过目前dart里的 `typedef` 只支持函数别名，不支持其他类型。

`typedef`甚至可以配合泛型使用，像下面这样

```dart
typedef Compare<T> = int Function(T a, T b);

int sort(int a, int b) => a - b;

void main() {
  assert(sort is Compare<int>); // True!
}
```

## 元数据 Metadata

dart里，可以使用注解方式，给代码一些额外的信息。dart内置了两个注解 `@deprecated` 和 `@override`。

还可以自定义一个类来作为注解。可以在运行时通过反射来获取注解信息。

感觉和 `Java` 里的注解有点像，不知道是不是，等后面再看吧。



## 终于差不多看完了



嗯，最后还有些什么 **注释** 之类的，就先填过吧，大同小异，看了也忘了……



 