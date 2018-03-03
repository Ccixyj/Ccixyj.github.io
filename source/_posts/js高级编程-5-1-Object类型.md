---
title: Object类型
date: 2018-03-03 19:17:04
tags: js高级编程
---


Object类型是JS中使用最多的一个类型.Object的实例不具备多少功能,但对于应用程序中存储和传输数据而言,他们确实是非常理想的选择.

<!-- more -->

创建Object有两种方式.

```js
//1.使用new方式
var person = new Object();
person.name = "zhang san";
person.age = 29;

//2.使用对象字面量表示法.
var person = {
    name : "zhang san", //属性名也可以说使用字符串,等价与"name"
    age : 29
};
//留空花括号也可以

var person = {};
person.name = "张三";
person.age = 29
```
推荐使用对象字面量语法,语法要求代码少,并且给人以封装数据的感觉.对象字面量也是函数传递大量可选参数的首选方式.如:

```js

//字面量可选参数
function display(args){
    var output = "";
    if (typeof args.name == "string") {
        output += "Name is " + args.name + ";\n";
    }

    if (typeof args.age == "number") {  
        output += "Age is "+ args.age + "; \n"
    }

    console.log(output);
};

display({
    name : "zhang san",
    age : 29
}); 

display({
    name : "li si"
});
```
这种传递方式适合需要向函数传递大量可选蚕食的情形.一般来讲,明明参数虽然容易处理,但有多个参数下就会不够灵活.最好的处理方式是必须值使用命名参数,可选参数使用对象字面量来封装多个参数.

调用对象属性时一般使用点`.`表示法,Js中也可以使用方括号来访问对象的属性.

```js
console.log(person.name ); //zhang san
console.log(person["name"])//zhang san

var porpName = "name"; //使用一个变量
console.log(person[porpName])//zhang san
```

从功能上看这两种语法没有区别,但方括号的优点是可以使用变量来访问属性及属性名是关键字或保留字或者导致语法错误的字符,如属性名是`first name`中包含空格.除非必要,一般还是使用`.`表示法.











