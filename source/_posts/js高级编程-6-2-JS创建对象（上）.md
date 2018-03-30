---
title: JS创建对象（上）
date: 2018-03-31 00:12:59
tags: js高级编程
---

Object构造函数或对象字面量都可以用来创建单个对象,但会产生大量重复代码.为了解决这个问题,人们开始使用工厂模式的一种变体. 

<!-- more -->

### 工厂模式

工厂模式是软件工程领域一种广为人知的设计模式。这种模式创建具体对象的过程。考虑到在JS中无法创建类，开发人员就发明了一种函数。用函数来封装特定接口创建对象的细节，如下：
```js
 // 工厂模式
 function createPerson(name, age, job) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function () {
      console.log(this.name)
  }
  return o;
 }
 var p1 = createPerson("Nick", 29 , "Sofe Engine")
 var p2 = createPerson("John",24,"Director")
 p1.sayName() //Nick
 p2.sayName() //John
```
工厂模式虽然解决了创建多个对象的问题,但没有解决对象识别的问题(怎样知道一个对象的类型)。

### 构造函数模式

JS中可以用构造函数来创建对象。同时，也可以自定义构造函数来创建对象的属性和方法。

```js
function Person(name, age, job){
 this.name = name;
 this.o.age = age;
 this.o.job = job;
 this.sayName = function(){
     console.log(this.name)
 }
}
var p1 = Person("Nick", 29 , "Sofe Engine")
var p2 = Person("John",24,"Director")
```
这段代码与`createPerson()`有以下不同：

* 没有显示创建对象;
* 直接将属性赋给对象;
* 没有`return`语句

另外:函数名`Person()`首字母大写,非构造函数首字母小写（OOP语法）。
要使用`Person`实例,必须使用`new`操作符。这种方式调用构造函数会经历以下4个步骤：

1. 创建一个新对象;
2. 将构造函数的作用域赋值给新对象（`this`就指向了这个新对象）;
3. 执行构造函数中的代码。
4. 返回新对象。

这两个对象都有一个`constructor`属性，指向`Person`，如下：

```js
console.log(p1.constructor == Person) //true
console.log(p2.constructor == Person) //true
```

对象的属性最初是用来标识对象类型的。通过构造函数，`p1`，`p2`即是`Object`类型，也是`Person`类型。

> 构造函数定义在全局的对象（浏览器中为`windows`对象）中。

####  1.将全局对象当作函数

构造函数与其他函数唯一的区别就在于调用方式不同。当然，构造函数也是函数，不存在定义的特殊语法。任何函数，通过`new`来调用，就作为构造函数；所以`Person`可以以下方式调用：

```js
 //a.构造函数
 var p3 = new Person("nickoo",28,"PHPer");
 p3.sayName(); //nickoo

 //b.普通函数
 Person("JJhon",20,"Javaer"); //添加到windows对象
 window.sayName(); //JJhon

 //c.在另一个对象的作用域中调用
 var o = new Object();
 Person.call(o,"Kate",25,"Nurse")
 o.sayName();
```

##### a. 例子展示了构造函数的典型用法。
##### b. 例子展示了不用`new`操作符会出现什么结果：属性和方法都添加给了`windows`对象。所以在调用完后，可以通过`windows`对象调用。
##### c. 例子通过使用`call()/apply()`方法在某个特殊对象的作用域中（对象`o`）调用`Person()`函数。因此，`o`就有了所有属性和`sayName()`方法。

#### 2. 构造函数的问题

构造函数的问题，就在于每个方法都要在每个实例上重新创建一遍。想想:函数是对象，因此创建对象时，函数也被创建一次。以下代码可以证明：

```js
console.log(p1.sayName == p2.sayName) //false
```


### 原型模式

每个函数都有一个`prototype`属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法。使用原型的好处是可以让所有对象的实例共享它的属性何方法。

```js
 function PersonPro() {
 }

 PersonPro.prototype.name = "Nick_Pro";
 PersonPro.prototype.age = 29;
 PersonPro.prototype.job = "Java_Pro";
 PersonPro.prototype.sayName = function () {
  console.log(this.name)
 }
 var per1 = new PersonPro()
 per1.sayName() //Nick_Pro


 var per2 = new PersonPro()
 per1.sayName() //Nick_Pro
```
#####  1. 理解原型对象

无论什么时候，只要创建了一个新函数，就会为该函数创建一个`prototype`属性，这个属性指向函数的原型对象。所有原型对象都会获得一个`constructor`属性，属性包含一个指向`prototype`属性的函数指针。
创建自定义函数后，其原型默认只会取得`constructor`属性;至于其他对象，都是从`Object`继承而。，虽然在所有实现都无法访问`[[prototype]]`,但可以通过`isPrototypeOf()`来确定对象之间是否存在关系。

```js
 console.log(PersonPro.prototype.isPrototypeOf(per1))
 console.log(PersonPro.prototype.isPrototypeOf(per2))
  //ECMA 新增的Object.getPrototype() 
console.log(Object.getPrototypeOf(per1))
console.log(Object.getPrototypeOf(per2).name)
```

使用`Object.getPrototypeOf()`可以方便的取得一个对象的原型，这在利用原型实现继承的情况下是非常重要的。虽然可以通过对象实例访问保存在原型的值，但却不能通过对象实例重写原型中的值。如：

```js
var per3 = new PersonPro();
per3.name = "hello"
console.log(per1.name)  // Nick_Pro 原型
console.log(per3.name)  //hello 实例

per3.name = null;
console.log(per3.name)  // 实例

delete per3.name        //delete 断开联系
console.log(per3.name)  // Nick_Pro 原型
```

通过`hasOwnProperty()`，可以知道属性是否是实例属性or原型属性。

> `Object.getOwnPropertyDescriptor()`只能用于实例属性。要取得原型描述符，必须在原型对象上调用`Object.getOwnPropertyDescriptor()`方法。

##### 2. 原型与`in`操作符

`in`操作符有两种方式。单独使用和在`for-in`中使用。单独使用，`in`操作符会在通过对象能够访问给定属性时返回`true`，无论属性存在于实例中还是原型中。

```js
var p1 = new PersonIN()
var p2 = new PersonIN()

console.log(p1.hasOwnProperty("name")) //false
console.log("name" in p1) //true
console.log("name" in p2) //true

p1.name = "jhon"
console.log("name" in p1) //true
```
使用`for-in`循环时，返回的是能够通过对象访问的、可枚举的（`enumrated`）属性，包括存在属性和原型中。屏蔽了原型中不可枚举的属性也会在枚举中出现。
```js
 Object.defineProperty(p1, "age", {
     value: 18,
     enumerable: false
 })
 console.log("age" in p1) //true
 for (let p in p1) {
     console.log(`prop : ${p}`) //name , sayName
 }

 console.log(Object.keys(p1)) //["name"]
 console.log(Object.getOwnPropertyNames(p1)) //["name", "age"]
 console.log("================")
 console.log(Object.keys(p1.__proto__)) //["name", "sayName"]
 console.log(Object.getOwnPropertyNames(p1.__proto__)) //["constructor", "name", "sayName"]
```

##### 3. 更简单的原型语法

更常用的做法是用一个对象字面量来重写整个原型对象。

```js
var p2 = new PersonOut()
  PersonOut.prototype = {
      name:"Nick",
      age:19,
      job:"Java",
      sayName:function(){
          console.log(this.name)
      }
  }
```
虽然最终结果相同，但`constructor`属性不再指向`Person`了。每创建一个对象，就创建它的`prototype`
对象，这个对象获得`constructor`属性。而我们完全重写了默认的`prototype`对象，因此，`constructor`属性也指向了新对象的`constructor`属性。

```js
var po1 = new PersonOut()
console.log(po1 instanceof PersonOut) // true
console.log(po1.constructor == PersonOut) //falese
```

##### 4.原型的动态性

由于原型在找值过程中是一次搜索，因此我们对原型对象的修改立即能够从实例上反映出来，即使新创建对象后修改原型也是如此。

```js
var friend = new Person()
 Person.prototype.sayHi= function(){
  console.log("hi")
 }
 friend.sayHi() //hi
```

但是如果重写原型对象就不一样了。我们知道，创建构造函数时，会创建一个指向最初原型的`[[Prototype]]`指针，而把原型修改为另一个对象就等于切断了构造函数与最初原型的联系。

> 实例中的指针仅指向原型，而不指向构造函数。

```js
function PersonC(){};
var pc = new PersonC()
PersonC.prototype = {
 constructor:PersonC,
 name : 'nick',
 sayName :function(){
     console.log(this.name)
 }
};
pc.sayName() // Uncaught TypeError: pc.sayName is not a function
```

##### 5. 原生对象的原型

所有原生的引用类型都采用原型模式创建的。如：`Array`，`String`等，并在基础上定义了方法。如`Array`有`sort()`，`String`有`subString()`方法。使用者也可以自定义新的方法。

> 尽管可以这么做，但不推荐。在原生对象中添加方法，这样会导致可能的命名冲突，也可能意外导致重写原生方法。参阅：[JavaScript 社区由一个库引发的“smoosh门”事件到底怎么回事？](https://juejin.im/post/5ab0a7c06fb9a028cb2d7550)

##### 6. 原型对象的问题

原型模式省略了构造函数出始化环节，所以所有实例在默认情况下都是相同属性。原型属性最大的问题是由其共享本质所导致的。

原型中的属性被所有实例共享，对于函数非常合适。对于包含引用类型的属性来说，就非常不合适了。
```js
function PersonArray(){};
PersonArray.prototype.name = "nick"
PersonArray.prototype.friends = ["bob","dlean"]

var pa1 = new PersonArray();
var pa2 = new PersonArray();

pa1.friends.push("Vani"); 
console.log(pa1.friends); //["bob", "dlean", "Vani"]
console.log(pa2.friends); //["bob", "dlean", "Vani"]
console.log(pa1.friends === pa2.friends); //true
```

### 组合使用构造函数模式和原型模式

创建自定义类型的常见方式,就是组合使用函数模式和原型模式.构造函数使用自定义实例属性,原型模式用于定义方法和共享的属性。这种模式支持向构造函数传递参数。
```js
function PersonFinal(name, age) {
    this.name = name
    this.age = age
    this.friends = ["shely", "yang"]
}
PersonFinal.prototype = {
    constructor: PersonFinal,
    sayName: function () {
        console.log(this.name)
    }
}

var pf1 = new PersonFinal("zane", 22)
var pf2 = new PersonFinal("Dom", 92)

pf1.friends.push("cindy")
console.log(pf1.friends) //["shely", "yang", "cindy"]
console.log(pf2.friends) //["shely", "yang"]
console.log(pf1.sayName == pf2.sayName) //true

```
这种构造函数与原型混成的模式是JS中最广泛与认同度最高的一种。可以说用来自定义的一种默认模式。


