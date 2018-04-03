---
title: JS创建对象（下）
date: 2018-04-03 17:09:35
tags: js高级编程
---

有的开发人员在看到独立的构造函数和原型时，会感到困惑。动态原型正是致力于解决这一个问题的方案，把所有的信息封装在构造函数中，而通过在构造函数中初始化原型（仅在必要情况下），又保持了构造函数和原型的优点。可以通过检查某个应该存在的方法是否有效，来决定是否需要初始化原型。

<!-- more -->
    
### 动态原型模式

```js
//动态原型模式
 function Person(name , age ){
  this.name = name ;
  this.age = age;
  if(typeof this.sayName != 'function'){
   Person.prototype.sayName = function(){
       console.log(this.name);
   };
  }
 }
 var frined = new Person("Rock",18)
 frined.sayName() //Rock

```
注意方法添加部分。构造函数只在`sayname`不存在时才添加到原型中去。并且只在初次调用时才会执行。此后，原型已经初始化完成，不再修改。这里对原型的修改，能够立即在实例中得到反映。采用这种方法创建的对象，可以使用`instanceof`操作符确定它的类型。

> 使用动态类型时，不能使用字面量重写原型。如果重写，会切断现有实例与新原型之间的联系。



### 寄生构造函数模式

通常，在前几种模式都不适用的情况下，可以使用寄生构造函数模式。这种模式的基本思想是创建一个函数，该函数的作用仅仅封装创建对象的代码，然后返回新创建的对象；但从表面上看，这是函数像是典型的构造函数。
```js
//寄生构造模式O
function PersonP(name , age){
    var o = {};
    o.name = name ;
    o.age = age;
    o.sayName = function (){
        console.log(this.name)
    }
    return o ;
};

var frinedO = new PersonP("doom",10000)
frinedO.sayName() //doom
```

这个模式可以在特殊的情况下来为对象创建一个具有额外方法的特殊数组。由于不能直接修改`Array`构造函数，因此可以使用这个模式。

缺点：返回的对象与构造函数的原型属性之间没有关系，为此，不能使用`instanceOf`操亲确定对象类型。

### 稳妥构造模式

稳妥对象，指的是没有公共属性，而且其方法也不引用`this`对象。稳妥对象适合在一些安全环境中（这些环境会禁止使用`this`和`new`），或者防止数据被其他应用程序改动时使用。稳妥构造函数与寄生构造函数类似，但有两点不同：一个是创建对象的实例方法不引用`this`;第二个不使用`new`操作符调用构造函数。

```js
 function PersonS(name ,age){
  var o = {};
  //添加私变量和函数
  Object.defineProperty(o,"name",{
   value:name,
   configurable:false,
  
  })
  o.sayName  = function(){
   console.log(name)
  };
  return o;
 }
 //注意，除了使用sayName()方法外，没有其访问
 var po = PersonS("ricqiu","20")
 po.sayName() 
```
> 构造模式与寄生模式类似，构造的对象也和构造函数之间没什么关系，`instanceof`操作符对这种对象也没什么意义。
