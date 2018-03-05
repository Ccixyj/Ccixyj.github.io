---
title: Array类型
date: 2018-03-05 16:56:01
tags: js高级编程
---


除了Object外,Array类型恐怕是最常用的类型了.Js中数组不但是有序列表,且每一项可以保存任何数据类型.Js的数组大小是可以动态调整的,即可以随着数据的添加自动增长以容纳新数据.

<!-- more -->

创建数组的基本方式有两种.
第一种方法:使用Array的构造函数.

```js
var colors = new Array();

//如果知道长度,也可以预先传递长度
var colors = new Array(20);

//也可以传递数组初始化
var colors = new Array("red", "blue", "green");

//可以省略new关键字
var colors = Array(3) //长度为3的数组
var names = Array("json") //创建包含一项,即字符串json 
```
第二种方法:使用数组字面量表示法.数组字面量由一对包含数组的方括号表示,多个数组直接以逗号分割.
```js
//字面量
var colors = ["red", "blue", "green"];
var names = [] //创建一个空数组

var values = [1,2,] //不要这样,会创建一个包含2或3项的数组
var ops = [,,,,,] //不要这样,会创建包含5或6项的数组
```

读取和设置数组的值时,要使用方括号的索引值.
```js
var colors = ["red", "blue", "green"];
console.log(colors[0]); //打印第一项
colors[2] = "black"  //修改第三项
colors[3] = "brown"  //新增第四项
```

数组的`length`很有特点----它不是只读的.可以通过设置这个属性,从数组的末尾移除或者添加新的项.

```js
//lenth
var colors = ["red", "blue", "green"];
colors.length = 2
console.log(colors[2]) //undefined

var colors = ["red", "blue", "green"];
colors.length = 5
console.log(colors[4]) //undefined
```
一开始`colors`有三项,将`length`设置为2后,会移除最后一项,在访问就是`undefined`了.如果这种大于长度的值,则会扩容到`length`的长度,且每一项都为`undefined`.

利用`length`可以方便的在末尾添加新项.设置了`length`后,会重新计算`length`.

```js
var colors = ["red", "blue", "green"];
colors[colors.length] = "black" //添加'black' ,index 为3 
colors[colors.length] = "white" //添加'white' ,index 为4
```
>[点击此处查看具体文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)


### 检测数组

确定某个对象是不是数组的经典问题.
如果是一个网页或者全局作用域,使用`instanceof`操作符就能得到结果,.但是如果网页中包含多个和框架,即有多个全局环境,从而存在多个Array构造函数.如果从一个框架传入另一个框架,那么就有两个不同的构造函数.

为了解决这个问题,ES5新增了`Array.isArray()`方法.它可以解决上述问题.

### 栈方法

数组的行为可以表现的向栈一样.栈是一种**LIFO(Last-In-First-Out,后进先出)**的数据结构,最先加入的项最早被移除.栈中的插入*(推入:`push())`*和移除*(弹出`pop()`)*只发生在栈的顶部.如下:

```js
//push pop
var colors = new Array();

var count = colors.push("red" , "blue");
console.log(count); //2

var count = colors.push("black");
console.log(count); //3

var item = colors.pop()
console.log(item); //black
console.log(colors.length); //2
```

### 队列方法

数组还是一种队列的数据结果,其访问规则是**FIFO(Fisrt-IN-First-Out,先进先出)**.队列在末尾添加项,在前端移除项.前端移除项的方法为`shift`,它能够移除数组的第一项,同时将长度减一.

```js
//shift
var colors = new Array();

var count = colors.push("red" , "blue");
console.log(count); //2

var count = colors.push("black");
console.log(count); //3

var item = colors.shift()
console.log(item); //red
console.log(colors.length); //2
```

### 排序方法

1.`reverse()`方法会翻转数组的顺序.

```js
//sort
var values = [0,1,5,10,15]
values.reverse()
console.log(values) //[15,10,5,1,0]
```

2.`sort()`方法:按升序排列数组.

```js
//sort
var values = [0,1,5,10,15]
values.sort()
console.log(values) //[0, 1, 10, 15, 5]
```
发现在进行的是字符串比较.不服和需求.所有`sort()`可以接收一个比较参数.

```
function compare(a,b){
    if(a<b) return -1;
    if(a>b) return  1;
    return 0;
}

values.sort(compare)
console.log(values) //[0, 1, 10, 15, 5]
```

如果是降序,则叫魂比较函数的返回值即可.

```js
function compare(a,b){
    if(a<b) return 1;
    if(a>b) return -1;
    return 0;
}

values.sort(compare)
console.log(values) //[15, 10, 5, 1, 0]
```

### 操作方法

1.`contact()`方法:基于当前数组的所有项创建一个新数组.源数组保持不变.

2.`slice()`方法:基于当前的一项或多项创建一个新的数组.该方法不会影响原数组.例如:
```js
//sclice
var colors = ["red","green","blue","yellow","purple"]
var colors2 = colors.slice(1) //["green","blue","yellow","purple"]
var colors3 = colors.slice(1,4) //["green","blue","yellow"]
```

3.`splice()`方法,他有很多种用法.

 - 删除:可以删除任意项.指定两个参数:删除的第一项的位置和删除的项数.`splice(0,2)`会删除前两项.
 - 插入:可以向指定位置插入任意数量的项,需提供3个参数.第二个参数指定为0.
 - 替换:可以向指定位置插入任意数量的项,需提供3个参数.需设置第二个参数删除的项.
 

### 迭代方法

- every()
- filter()
- foreach()
- map()
- some()

### 缩小方法

- reduce()
- reduceRight()