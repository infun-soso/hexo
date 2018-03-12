---
title: "对于js中原型与原型链的理解"
date: 2018-1-22 14:21:10
categories: Js相关
tags: [prototype, 原型链，原型]
---

<img src="/314601.jpg" width="100%">

### 前言

在JavaScript中，原型也是一个对象，通过原型可以实现对象的属性继承，JavaScript的对象中都包含了一个”[[Prototype]]”内部属性，这个属性所对应的就是该对象的原型。
“[[Prototype]]”作为对象的内部属性，是不能被直接访问的。所以为了方便查看一个对象的原型，Firefox和Chrome中提供了__proto__这个非标准（不是所有浏览器都支持）的访问器（ECMA引入了标准对象原型访问器”Object.getPrototype(object)”）。在JavaScript的原型对象中，还包含一个”constructor”属性，这个属性对应创建所有指向该原型的实例的构造函数
<!--more-->


### 普通对象与函数对象

JavaScript 中，万物皆对象！但对象也是有区别的。分为普通对象和函数对象，Object 、Function 是 JS 自带的函数对象。下面举例说明
```
var o1 = {}; 
var o2 =new Object();
var o3 = new f1();

function f1(){}; 
var f2 = function(){};
var f3 = new Function('str','console.log(str)');

console.log(typeof Object); //function 
console.log(typeof Function); //function  

console.log(typeof f1); //function 
console.log(typeof f2); //function 
console.log(typeof f3); //function   

console.log(typeof o1); //object 
console.log(typeof o2); //object 
console.log(typeof o3); //object

```

注意：Function Object 也都是通过 New Function()创建的。
### 构造函数

**实例的构造函数属性（constructor）是一个指针，它指向构造函数。**

```
function Person(name, age, job) {
 this.name = name;
 this.age = age;
 this.job = job;
 this.sayName = function() { alert(this.name) } 
}
var person1 = new Person('Zaxlct', 28, 'Software Engineer');
var person2 = new Person('Mick', 23, 'Doctor');
```
即
```
person1.constructor = Person
person2.constructor = Person
```
###  原型对象

在javaScript中，每当定义一个对象（函数也是对象）的时候，对象中都会包含一些预定的属性，其中每个**函数对象**都有一个==prototype==属性，这个属性指向函数的原型对象，（先不用管什么是__proto__后面会详细的剖析）

```
function Person() {}
Person.prototype.name = 'Zaxlct';
Person.prototype.age  = 28;
Person.prototype.job  = 'Software Engineer';
Person.prototype.sayName = function() {
  alert(this.name);
}
  
var person1 = new Person();
person1.sayName(); // 'Zaxlct'

var person2 = new Person();
person2.sayName(); // 'Zaxlct'

console.log(person1.sayName == person2.sayName); //true

```
> 每个对象都有__proto__属性，但只有函数对象有prototype属性

那什么是**原型对象**呢？
我们把上面的例子改一改，你就会明白了：

```
Person.prototype = {
   name:  'Zaxlct',
   age: 28,
   job: 'Software Engineer',
   sayName: function() {
     alert(this.name);
   }
}
```
原型对象，顾名思义，他就是一个普通对象。从现在开始你要牢牢记住原型对象就是Person.prototype，如果你还是害怕它，那就把它想成一个字母A：var A = Person.prototype

在上面我给A添加了四个属性:name age job sayName。其实它还有一个默认的属性:constructor

>在默认情况下，所有的原型对象都会自动获得一个constructor（构造函数）属性，这个属性（是一个指针）指向prototype属性所在的函数(Person)

上面这句话有点拗口，我们翻译一下：A有一个默认的constructor属性，这个属性是一个指针，指向Person。即：

==Person.prototype.constructor == Person==

在上面构造函数里，我们知道实例的构造函数属性指向构造函数

==person1.constructor == Person==

这两个公式好像有点联系：

> person1.constructor == Person
> 
> Person.prototype.constructor == Person

person1 为什么有 constructor 属性？那是因为 person1 是 Person 的实例。
那 Person.prototype 为什么有 constructor 属性？？同理， Person.prototype （你把它想象成 A） 也是Person 的实例。
也就是在 Person 创建的时候，创建了一个它的实例对象并赋值给它的 prototype，基本过程如下：

```
var A = new Person()
Person.prototype = A
```
**在对象被创建时，会产生一个新的对象Person.prototype，实例并没有constructor属性，所谓的person1.constructor其实是访问Person.prototype的constructor属性，都指向构造器本身。**

###  __proto __

JS 在创建对象（不论是普通对象还是函数对象）的时候，都有一个叫做__proto__ 的内置属性，用于指向创建它的构造函数的原型对象。
对象 person1 有一个 __proto__属性，创建它的构造函数是 Person，构造函数的原型对象是 Person.prototype ，所以：
person1.proto== Person.prototype

> 不过，要明确的真正重要的一点就是，这个连接存在于实例（person1）与构造函数（Person）的原型对象（Person.prototype）之间，而不是存在于实例（person1）与构造函数（Person）之间。

###  构造器

我们可以这样创建一个对象

var obj = {}

等同于

var obj = new Object()

obj是构造函数Object的一个实例，所以：

obj.constructor = Object

obj.__proto__ = Object.prototype


同理，可以创建对象的构造器不仅仅有Object，也可以是Array，Date，Function等
```
var b = new Array();
b.constructor === Array;
b.__proto__ === Array.prototype;

var c = new Date(); 
c.constructor === Date;
c.__proto__ === Date.prototype;

var d = new Function();
d.constructor === Function;
d.__proto__ === Function.prototype;

```

###  总结

1. 所有函数对象的proto都指向Function.prototype，他是一个空函数
2. 类似Array.prototype一般为普通对象(Array.prototype.proto == Object.prototype)，但是Function.prototype为函数对象
3. Function.prototype.proto == Object.prototype
4. Object.prototype.proto == null 顶层
5. console.log(Array.prototype) 空数组
6. 函数对象的constructor指向函数本身

























