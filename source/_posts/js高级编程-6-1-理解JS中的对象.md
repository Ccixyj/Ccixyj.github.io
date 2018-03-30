---
title: 理解JS中的对象
date: 2018-03-16 00:07:46
tags: js高级编程
---

JS中的对象可以看作是散列表:有键值对组成,值可以是数据或函数.每个对象都是基于一个引用类型创建,这个类型可以是原生类型,也可以是自定义的类型.

<!-- more -->

创建对象的属性和方法如下:
```js
//创建对象
var person = new Object();
person.name = "nick";
person.age = 29;
person.sayName = function(){
    console.log(this.name);
}

//更多的使用字面量创建对象
var person ={
    name : "nick",
    age : 29,
    sayName : function(){
        return this.name;
    }
}
```

### 属性类型 ###

ECMA-262定义只有内部才用的特性(attribute)时,描述了属性(property)的各种特征.ECMA-262定义这些特性是为JS引擎用的,JS中不能直接访问.为了表示这些是内部值,该规范把它们放在了两对方括号中,例如`[[Enumerable]]`.

ECMAScript中有两种属性:数据属性和访问属性.

#### 1.数据属性 ####

数据属性包含一个数据值的位置.在这个位置可以读取和写入.数据属性有4个特性:
要修改属性的默认特性,必须使用`Object.defineProperty()`方法.[ 参考文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

* `[[Configurable]]`: 表示能否通过 `delete`删除属性从而重新定义属性,能否修改属性的特性,或者能否把属性修改为访问器属性.直接在对象上定义,默认为`true`,否则`false`
* `[[Enumerable]]`: 表示能否通过`for-in`循环返回属性.直接在对象上定义,默认为`true`,否则`false`
*  `[[Writable]]`: 表示能否修改属性的值.直接在对象上定义,默认为`true`,否则`false`
*  `[[Value]]`: 包含这个属性的数据值.读取属性值的时候,从这个位置读;写入属性的时候,把新值保存在这个位置.默认`undefined`.

对于前面的`person`对象,它的`[[Configurable]]`,`[[Enumerable]]`,`[[Writable]]`都为`true`,`name`属性的 `[[Value]]`特性被设置为`"nick"`.对这个值的修改都将反应在这个位置.

```js
//Object.defineProperty() 
//Configurable  Enumerable Writable默认都为false
var o = {};
Object.defineProperty(o,'a',{
    value:1
})
o.a = 2;             //设置a的值,无效
console.log(o.a);    // 1
delete o.a;          //false
for(let p in o){
    console.log(p)   //empty
}
```

#### 2.访问器属性 ####

访问器属性有四个.

*  `[[Configurable]]`: 表示能否通过 `delete`删除属性从而重新定义属性,能否修改属性的特性,或者能否把属性修改为访问器属性.直接在对象上定义,默认为`true`,否则`false`
*  `[[Enumerable]]`: 表示能否通过`for-in`循环返回属性.直接在对象上定义,默认为`true`,否则`false`
*  `[[Get]]`:在读取属性时调用的函数.默认为`undefined`
*  `[[Set]]`在写入属性时调用的函数.默认为`undefined`

访问器属性不能直接定义,必须使用`Object.defineProperty()`来定义.

```js
//访问器属性
var book = {
    _year:2004,
    edition:1
};

Object.defineProperty(book,"year",{
    get:function(){return this._year; },
    set:function(newVar){
        this._year = newVar;
        this.edition += newVar - 2004;
    }
})

book.year = 2005; 
console.log(book.edition); //2
```

如果只指定`getter`意味着属性是只读的.严格模式下,写入`getter`属性会报错.如果只指定`setter`也不能读,非严格模式下返回`undefined`,在严格模式下返回错误.

另外有非标准的`obj.__defineSetter__(prop, fun)` 和 `obj.__defineGetter__(prop, func)`.**该特性是非标准的，请尽量不要在生产环境中使用它！,且为已废弃状态**

###  定义多个属性 ###

由于定义多个属性的可能性很大,所以又定义了`Object.defineProperties(obj, props)`方法.

```js
//定义多个属性
var book = {} ;
Object.defineProperties(book,{
    _year:{
        writable : true, //可写
        value:2004
        },
    edition:{
        writable : true, //可写
        value:1
        },
    year:{
        get:function(){
            return this._year ;
        },
        set:function(newVar){
            if(newVar > 2004){
                this._year = newVar;
                this.edition += newVar - 2004;
            }
        }
    }
})
book.year = 2005; 
console.log(book.edition); //2
```

### 读取属性的特性 ###

`Object.getOwnPropertyDescriptor(obj, prop)` 方法返回指定对象上一个自有属性对应的属性描述符。（自有属性指的是直接赋予该对象的属性，不需要从原型链上进行查找的属性）

```js
var o, d;

o = { get foo() { return 17; } };
d = Object.getOwnPropertyDescriptor(o, "foo");
// d {
//   configurable: true,
//   enumerable: true,
//   get: /*the getter function*/,
//   set: undefined
// }

o = { bar: 42 };
d = Object.getOwnPropertyDescriptor(o, "bar");
// d {
//   configurable: true,
//   enumerable: true,
//   value: 42,
//   writable: true
// }

o = {};
Object.defineProperty(o, "baz", {
  value: 8675309,
  writable: false,
  enumerable: false
});
d = Object.getOwnPropertyDescriptor(o, "baz");
// d {
//   value: 8675309,
//   writable: false,
//   enumerable: false,
//   configurable: false
// }
```