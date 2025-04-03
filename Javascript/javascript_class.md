#  javascript class 

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/7c3f9559842ea2fcfa61345b4b2ded7f.png)



##  1. 简介
**类是用于创建对象的模板。**

我们使用 class 关键字来创建一个类，类体在一对大括号 {} 中，我们可以在大括号 {} 中定义类成员的位置，如方法或构造函数。

每个类中包含了一个特殊的方法 `constructor()`，它是类的构造函数，这种方法用于创建和初始化一个由 class 创建的对象。

创建一个类的语法格式如下：

```bash
class ClassName {
  constructor() { ... }
}
```
实例

```bash
class User {

  constructor(name) {
    this.name = name;
  }

  sayHi() {
    alert(this.name);
  }

}

// Usage:
let user = new User("John");
user.sayHi();
```

## 2. 浏览器支持
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d082e09d3a02cdc2f3a0b2e002e3f0b.png)
##  3. 使用类
定义好类后，我们就可以使用 `new` 关键字来创建对象：

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript 类</h2>

<p>如何使用 JavaScript 类</p>

<p id="demo"></p>

<script>
class Runoob {
  constructor(name, url) {
    this.name = name;
    this.url = url;
  }
}
 
let site = new Runoob("菜鸟教程",  "http://www.runoob.com");
document.getElementById("demo").innerHTML =
site.name + "：" + site.url;
</script>

</body>
</html>
```
输出：

```bash
JavaScript 类
如何使用 JavaScript 类

菜鸟教程：http://www.runoob.com
```
创建对象时会自动调用构造函数方法 `constructor()`。

## 4. 类表达式
类表达式是定义类的另一种方法。类表达式可以命名或不命名。命名类表达式的名称是该类体的局部名称。

```bash
// 未命名/匿名类
let Runoob = class {
  constructor(name, url) {
    this.name = name;
    this.url = url;
  }
};
console.log(Runoob.name);
// output: "Runoob"
 
// 命名类
let Runoob = class Runoob2 {
  constructor(name, url) {
    this.name = name;
    this.url = url;
  }
};
console.log(Runoob.name);
// 输出: "Runoob2"
```
构造方法

构造方法是一种特殊的方法：

 - 构造方法名为 constructor()。
 - 构造方法在创建新对象时会自动执行。
 - 构造方法用于初始化对象属性。
 - 如果不定义构造方法，JavaScript 会自动添加一个空的构造方法。

## 5. 类的方法
我们使用关键字 `class` 创建一个类，可以添加一个 `constructor()` 方法，然后添加任意数量的方法。

```bash
class ClassName {
  constructor() { ... }
  method_1() { ... }
  method_2() { ... }
  method_3() { ... }
}
```
以下实例我们创建一个 "age" 方法，用于返回网站年龄：

```bash
class Runoob {
  constructor(name, year) {
    this.name = name;
    this.year = year;
  }
  age() {
    let date = new Date();
    return date.getFullYear() - this.year;
  }
}
 
let runoob = new Runoob("菜鸟教程", 2018);
document.getElementById("demo").innerHTML =
"菜鸟教程 " + runoob.age() + " 岁了。";
```

## 6. 严格模式 "use strict"
类声明和类表达式的主体都执行在严格模式下。比如，构造函数，静态方法，原型方法，`getter` 和 `setter` 都在严格模式下执行。

如果你没有遵循严格模式，则会出现错误：

```bash
class Runoob {
  constructor(name, year) {
    this.name = name;
    this.year = year;
  }
  age() {
    // date = new Date();  // 错误
    let date = new Date(); // 正确
    return date.getFullYear() - this.year;
  }
}
```
## 7. 类关键字

### 7.1 extends
extends 关键字用于创建一个类，该类是另一个类的子类。

子类继承了另一个类的所有方法。

继承对于代码可重用性很有用：在创建新类时重用现有类的属性和方法。

`super()` 方法引用父类的构造方法。

通过在构造方法中调用 super() 方法，我们调用了父类的构造方法，这样就可以访问父类的属性和方法。

语法

```bash
class childClass extends parentClass
```
实例
```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript 类继承</h2>

<p>JavaScript 类继承使用 extends 关键字。</p>
<p>"super" 方法用于调用父类的构造函数。</p>

<p id="demo"></p>

<script>
class Site {
  constructor(name) {
    this.sitename = name;
  }
  present() {
    return '我喜欢' + this.sitename;
  }
}
 
class Runoob extends Site {
  constructor(name, age) {
    super(name);
    this.age = age;
  }
  show() {
    return this.present() + ', 它创建了 ' + this.age + ' 年。';
  }
}
 
let noob = new Runoob("菜鸟教程", 5);
document.getElementById("demo").innerHTML = noob.show();
</script>

</body>
</html>
```
输出：

```bash
JavaScript 类继承
JavaScript 类继承使用 extends 关键字。

"super" 方法用于调用父类的构造函数。

我喜欢菜鸟教程, 它创建了 5 年。
```
### 7.2 static 
类（class）通过 static 关键字定义静态方法。

静态方法调用直接在类上进行，不能在类的实例上调用。

静态方法通常用于创建实用程序函数。

语法

```bash
static methodName()
```


```bash
class Runoob {
  constructor(name) {
    this.name = name;
  }
  static hello() {
    return "Hello!!";
  }
}
 
let noob = new Runoob("菜鸟教程");
 
// 可以在类中调用 'hello()' 方法
document.getElementById("demo").innerHTML = Runoob.hello();
 
// 不能通过实例化后的对象调用静态方法
// document.getElementById("demo").innerHTML = noob.hello();
// 以上代码会报错
```
### 7.3  super
super 关键字用于访问和调用一个对象的父对象上的函数。。

在构造函数中使用时，super关键字将单独出现，并且必须在使用 this 关键字之前使用。super 关键字也可以用来调用父对象上的函数。

实例

```bash
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript 类继承</h2>

<p>JavaScript 类继承使用 extends 关键字。</p>
<p>"super" 方法用于调用父类的构造函数。</p>

<p id="demo"></p>
<p id="demo2"></p>
<script>
class Polygon {
  constructor(height, width) {
    this.name = 'Rectangle';
    this.height = height;
    this.width = width;
  }
  sayName() {
    return 'Hi, I am a ', this.name + '.';
  }
  get area() {
    return this.height * this.width;
  }
  set area(value) {
    this._area = value;
  }
}
 
class Square extends Polygon {
  constructor(length) {
    // 这里，它调用父类的构造函数的，
    // 作为 Polygon 的 height, width
    super(length, length);
	  
	this.height; // 需要放在 super 后面，不然引发 ReferenceErro
 
    // 注意：在派生的类中，在你可以使用'this'之前，必须先调用 super()。
    // 忽略这，这将导致引用错误。
    this.name = 'Square';
  }
}
 
let s = new Square( 5 );
document.getElementById("demo").innerHTML = s.sayName();
document.getElementById("demo2").innerHTML = s.area;
</script>

</body>
</html>
```
输出：

```bash
JavaScript 类继承
JavaScript 类继承使用 extends 关键字。

"super" 方法用于调用父类的构造函数。

Square.

25
```
用 `super` 调用父类的静态方法：

```bash
!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>

<h2>JavaScript 类继承</h2>

<p>JavaScript 类继承使用 extends 关键字。</p>
<p>"super" 方法用于调用父类的静态方法。</p>

<p id="demo"></p>
<p id="demo2"></p>
<script>
class Rectangle {
  constructor() {}
  static logNbSides() {
    return 'I have 4 sides';
  }
}
 
class Square extends Rectangle {
  constructor() {}
  static logDescription() {
    return super.logNbSides() + ' which are all equal';
  }
}

document.getElementById("demo").innerHTML = Square.logDescription(); // 'I have 4 sides which are all equal'
</script>

</body>
</html>
```
输出：

```bash
JavaScript 类继承
JavaScript 类继承使用 extends 关键字。

"super" 方法用于调用父类的静态方法。

I have 4 sides which are all equal
```


