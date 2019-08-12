# [译]编写优雅的JavaScript代码 - 最佳实践



**[原文]**： [https://devinduct.com/blogpost/22/javascript-clean-code-best-practices](https://devinduct.com/blogpost/22/javascript-clean-code-best-practices) 



## 有没有似曾相识



如果你对于代码，除了关注是否能准确的执行业务逻辑，还关心代码本身是怎么写的，是否易读，那么你应该会关注如何写出**干净优雅**的代码。作为专业的工程师，除了保证自己的代码没有bug，能正确的完成业务逻辑，还应该保证几个月后的自己，或者其他工程师，也能够维护自己的代码。你写的每一段代码，通常情况下，都不会是 **一次性** 工作，通常伴随着后续的不断迭代。如果代码不够优雅，那么将来维护这段代码的人(甚至你自己)，都将感到非常痛苦。祈祷吧，将来面对这些糟糕代码的人，不是你自己，而是别人 😓。

OK，我们先来简单定义下，什么是 **干净优雅** 的代码：**干净优雅的代码，应该是自解释的，容易看懂的，并且很容易修改或者扩展一些功能** 。

现在，静下来回忆一下，有多少次，当你接手前辈留下来的糟糕代码而懵逼时，心里默默的说过 "我*"的：

* "我*，那是啥玩意儿"
* "我*，这段代码是干啥的”
* "我*，这个变量又是干啥的"

**[译注]**： 我*，作者真大神啊，上面描绘的太真实了。有木有一种 `Big brother is watching you` 的赶脚。

嗯，下面这个图片完美的展示了这种情形：

![](./1.jpeg)



引用 *Robert C. Martin* 的名言来说明这种情况：

> 丑陋的代码也能实现功能。但是不够优雅的代码，往往会让整个开发团队都跪在地上哭泣。

在这篇文章里，我主要讲下载 `JavaScript`里怎么书写干净优雅的代码，但是对于其他编程语言，道理也是类似的。



## JavaScript优雅代码的最佳实践



### 1. 强类型校验

使用 `===` 而不是 `==`  。

**[译注]** ：这一条应该是广泛接受了吧，居然还有人面试会问 `==` 类型转换的问题……

```javascript
// If not handled properly, it can dramatically affect the program logic. It's like, you expect to go left, but for some reason, you go right.
0 == false // true
0 === false // false
2 == "2" // true
2 === "2" // false

// example
const value = "500";
if (value === 500) {
  console.log(value);
  // it will not be reached
}

if (value === "500") {
  console.log(value);
  // it will be reached
}
```

### 2. 变量命名

变量、字段命名，应该包含它所对应的真实含义。这样更容易在代码里搜索，并且其他人看到这些变量，也更容易理解。

**错误的示范** 

```javascript
let daysSLV = 10;
let y = new Date().getFullYear();

let ok;
if (user.age > 30) {
  ok = true;
}
```

**正确的示范** 

```javascript
const MAX_AGE = 30;
let daysSinceLastVisit = 10;
let currentYear = new Date().getFullYear();

...

const isUserOlderThanAllowed = user.age > MAX_AGE;
```

不要在变量名中加入不必要的单词。

**错误的示范** 

```javascript
let nameValue;
let theProduct;
```

**正确的示范** 

```javascript
let name;
let product;
```

不要强迫开发者去记住变量名的上下文。

**错误的示范**

```javascript
const users = ["John", "Marco", "Peter"];
users.forEach(u => {
  doSomething();
  doSomethingElse();
  // ...
  // ...
  // ...
  // ...
  // Here we have the WTF situation: WTF is `u` for?
  register(u);
});
```

**正确的示范** 

```javascript
const users = ["John", "Marco", "Peter"];
users.forEach(user => {
  doSomething();
  doSomethingElse();
  // ...
  // ...
  // ...
  // ...
  register(user);
});
```

不要在变量名中添加多余的上下文信息。

**错误的示范** 

```javascript
const user = {
  userName: "John",
  userSurname: "Doe",
  userAge: "28"
};

...

user.userName;
```

**正确的示范** 

```javascript
const user = {
  name: "John",
  surname: "Doe",
  age: "28"
};

...

user.name;
```

### 3. 函数相关

尽量使用足够长的能够描述函数功能的命名。通常函数都会执行一个明确的动作或意图，那么函数名就应该是能够描述这个意图一个动词或者表达语句，包含函数的参数命名也应该能清晰的表达具体参数的含义。

**错误的示范** 

```javascript
function notif(user) {
  // implementation
}
```

**正确的示范** 

```javascript
function notifyUser(emailAddress) {
  // implementation
}
```

避免函数有太多的形参。比较理想的情况下，一个函数的参数应该 `<=`2个 。函数的参数越少，越容易测试。

**错误的示范** 

```javascript
function getUsers(fields, fromDate, toDate) {
  // implementation
}
```

**正确的示范** 

```javascript
function getUsers({ fields, fromDate, toDate }) {
  // implementation
}

getUsers({
  fields: ['name', 'surname', 'email'],
  fromDate: '2019-01-01',
  toDate: '2019-01-18'
});
```

如果函数的某个参数有默认值，那么应该使用新的参数默认值语法，而不是在函数里使用 `||` 来判断。

**错误的示范** 

```javascript
function createShape(type) {
  const shapeType = type || "cube";
  // ...
}
```

**正确的示范** 

```javascript
function createShape(type = "cube") {
  // ...
}
```

一个函数应该做一件事情。避免在一个函数里，实现多个动作。

**错误的示范** 

```javascript
function notifyUsers(users) {
  users.forEach(user => {
    const userRecord = database.lookup(user);
    if (userRecord.isVerified()) {
      notify(user);
    }
  });
}
```

**正确的示范** 

```javascript
function notifyVerifiedUsers(users) {
  users.filter(isUserVerified).forEach(notify);
}

function isUserVerified(user) {
  const userRecord = database.lookup(user);
  return userRecord.isVerified();
}
```

使用 `Object.assign` 来给对象设置默认值。

**错误的示范** 

```javascript
const shapeConfig = {
  type: "cube",
  width: 200,
  height: null
};

function createShape(config) {
  config.type = config.type || "cube";
  config.width = config.width || 250;
  config.height = config.width || 250;
}

createShape(shapeConfig);
```

**正确的示范** 

```javascript
const shapeConfig = {
  type: "cube",
  width: 200
  // Exclude the 'height' key
};

function createShape(config) {
  config = Object.assign(
    {
      type: "cube",
      width: 250,
      height: 250
    },
    config
  );

  ...
}

createShape(shapeConfig);
```

不要在函数参数中，包括某些标记参数，通常这意味着你的函数实现了过多的逻辑。

**错误的示范** 

```javascript
function createFile(name, isPublic) {
  if (isPublic) {
    fs.create(`./public/${name}`);
  } else {
    fs.create(name);
  }
}
```

**正确的示范** 

```javascript
function createFile(name) {
  fs.create(name);
}

function createPublicFile(name) {
  createFile(`./public/${name}`);
}
```

不要污染全局变量、函数、原生对象的 `prototype`。如果你需要扩展一个原生提供的对象，那么应该使用 `ES`新的 类和继承语法来创造新的对象，而 **不是** 去修改原生对象的`prototype` 。

**错误的示范** 

```javascript
Array.prototype.myFunc = function myFunc() {
  // implementation
};
```

**正确的示范** 

```javascript
class SuperArray extends Array {
  myFunc() {
    // implementation
  }
}
```

### 4. 条件分支

不要用函数来实现 **否定** 的判断。比如判断用户是否合法，应该提供函数 `isUserValid()` ，而 **不是** 实现函数 `isUserNotValid()` 。

**错误的示范** 

```javascript
function isUserNotBlocked(user) {
  // implementation
}

if (!isUserNotBlocked(user)) {
  // implementation
}
```

**正确的示范** 

```javascript
function isUserBlocked(user) {
  // implementation
}

if (isUserBlocked(user)) {
  // implementation
}
```

在你明确知道一个变量类型是 `boolean` 的情况下，条件判断使用 简写。这确实是显而易见的，**前提是你能明确这个变量是boolean类型，而不是 null 或者 undefined** 。

**错误的示范** 

```javascript
if (isValid === true) {
  // do something...
}

if (isValid === false) {
  // do something...
}
```

**正确的示范** 

```javascript
if (isValid) {
  // do something...
}

if (!isValid) {
  // do something...
}
```

在可能的情况下，尽量 **避免** 使用条件分支。优先使用 **多态** 和 **继承** 来实现代替条件分支。

**错误的示范** 

```javascript
class Car {
  // ...
  getMaximumSpeed() {
    switch (this.type) {
      case "Ford":
        return this.someFactor() + this.anotherFactor();
      case "Mazda":
        return this.someFactor();
      case "McLaren":
        return this.someFactor() - this.anotherFactor();
    }
  }
}
```

**正确的示范** 

```javascript
class Car {
  // ...
}

class Ford extends Car {
  // ...
  getMaximumSpeed() {
    return this.someFactor() + this.anotherFactor();
  }
}

class Mazda extends Car {
  // ...
  getMaximumSpeed() {
    return this.someFactor();
  }
}

class McLaren extends Car {
  // ...
  getMaximumSpeed() {
    return this.someFactor() - this.anotherFactor();
  }
}
```

### 5. ES的类

在ES里，类是新规范引入的语法糖。类的实现和以前 `ES5` 里使用 `prototype` 的实现完全一样，只是它看上去更简洁，你应该优先使用新的类的语法。

**错误的示范** 

```javascript
const Person = function(name) {
  if (!(this instanceof Person)) {
    throw new Error("Instantiate Person with `new` keyword");
  }

  this.name = name;
};

Person.prototype.sayHello = function sayHello() { /**/ };

const Student = function(name, school) {
  if (!(this instanceof Student)) {
    throw new Error("Instantiate Student with `new` keyword");
  }

  Person.call(this, name);
  this.school = school;
};

Student.prototype = Object.create(Person.prototype);
Student.prototype.constructor = Student;
Student.prototype.printSchoolName = function printSchoolName() { /**/ };
```

**正确的示范** 

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }

  sayHello() {
    /* ... */
  }
}

class Student extends Person {
  constructor(name, school) {
    super(name);
    this.school = school;
  }

  printSchoolName() {
    /* ... */
  }
}
```

使用方法的 `链式调用`。很多开源的JS库，都引入了函数的链式调用，比如 `jQuery` 和 `Lodash` 。链式调用会让代码更加简洁。在 `class` 的实现里，只需要简单的在每个方法最后都返回 `this`，就能实现链式调用了。

**错误的示范** 

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }

  setSurname(surname) {
    this.surname = surname;
  }

  setAge(age) {
    this.age = age;
  }

  save() {
    console.log(this.name, this.surname, this.age);
  }
}

const person = new Person("John");
person.setSurname("Doe");
person.setAge(29);
person.save();
```

**正确的示范** 

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }

  setSurname(surname) {
    this.surname = surname;
    // Return this for chaining
    return this;
  }

  setAge(age) {
    this.age = age;
    // Return this for chaining
    return this;
  }

  save() {
    console.log(this.name, this.surname, this.age);
    // Return this for chaining
    return this;
  }
}

const person = new Person("John")
    .setSurname("Doe")
    .setAge(29)
    .save();
```

### 6. 避免冗余代码

通常来讲，我们应该避免重复写相同的代码，不应该有未被用到的函数或者死代码(永远也不会执行到的代码)的存在。

我们太容易就会写出重复冗余的代码。举个栗子，有两个组件，他们大部分的逻辑都一样，但是可能由于一小部分差异，或者临近交付时间，导致你选择了把代码拷贝了一份来修改。在这种场景下，要去掉冗余的代码，只能进一步提高组建的抽象程度。

至于死代码，正如它名字所代表的含义。这些代码的存在，可能是在你开发中的某个阶段，你发现某段代码完全用不上了，于是就把它们放在那儿，而没有删除掉。你应该在代码里找出这样的代码，并且删掉这些永远不会执行的函数或者代码块。我能给你的惟一建议，就是当你决定某段代码再也不用时，就立即删掉它，否则晚些时候，可能你自己也会忘记这些代码是干神马的。

当你面对这些死代码时，可能会像下面这张图所描绘的一样：

![](./2.png)



## 结论



上面这些建议，只是一部分能提升你代码的实践。我在这里列出这些点，是工程师经常会违背的。他们或许尝试遵守这些实践，但是由于各种原因，有的时候也没能做到。或许当我们在项目的初始阶段，确实很好的遵守了这些实践，保持了干净优雅的代码，但是随着项目上线时间的临近，很多准则都被忽略了，尽管我们会在忽略的地方备注上 `TODO` 或者 `REFACTOR` (**但正如你所知道的，通常 `later`也就意味着`never`**)。

OK，就这样吧，希望我们都能够努力践行这些最佳实践，写出 **干净优雅**  的代码 ☺️













