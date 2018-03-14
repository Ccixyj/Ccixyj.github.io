---
title: Function类型
date: 2018-03-13 23:18:06
tags: js高级编程
---

说起来ECMAScript中什么最有意思,莫过于函数了----而有意思的根源,在于函数实际上是对象.每个函数都是`Function`的实例,而且与其他引用类型一样具有属性和方法.由于函数是对象,所以函数名实际上是一个指向函数的指针,不会与某个函数绑定.

<!-- more -->

```js
//函数的声明
function sum(num1 , num2){
    return num1 + num2;
}

//与上面类似的声明
var sum = function(num1 , num2){
    return num1 + num2;
};

//使用Function构造函数
//不推荐
var sum = new Function("num1","num2","return num1 + num2;") 
```
### 没有重载 ###
 
将函数名想象成指针,有助于理解JS为什么没有重载的概念.同名的函数,后面声明的会覆盖前面声明的函数.

### 作为值的函数 ###

因为函数名本身就是变量,所以函数可以作为值来使用.函数不仅可以作为传递参数也可以作为函数的结果返回.

### 函数内部属性 ###

在函数内部,有两个特殊对象:`arguments`和`this`.

`arguments`是一个类数组对象,包含着传入的所有参数.虽然`arguments`的主要用途是保存函数参数,但这个对象还有一个名叫`callee`的属性,该属性是一个指针,指向`arguments`对象的函数.

```js
function factorial(num){
    if(num <= 1) return 1;
    else return num * factorial(num-1);
}
```
以上方法如果函数名`factorial`,则无法得到正确的结果,所以可以这样修改:
```js
//消除函数名的耦合
function factorial(num){
    if(num <= 1) return 1;
    else return num * arguments.callee(num -1)
}

//测试
var trueFunc = factorial;
factorial = function(){ return 0; };

console.log(trueFunc(5))  //120
console.log(factorial(5)) //0
```

函数内部另一个特殊对象是`this`.`this`引用的是函数据以执行的环境对象----或者说是`this`的值.(全局作用域是`window`对象).

```js
//this
window.color = 'red';
var o = {color:'blue'};

function sayColor(){
    return console.log(this.color);
}
sayColor()   //red

o.sayColor = sayColor;
o.sayColor() //blue
```
> 注意:函数的名字只是一个包含指针的变量而已.因此,即使在不同的执行环境中,全局的`sayColor()`函数与`o.sayColor`指向的仍是同一个函数.

### 函数的属性和方法 ###

JS中的函数是对象,所以函数也有属性和方法.每个函数包含两个属性:`length`和`prototype`.

`length`属性表示函数希望接收的命名参数个数.
`prototype`是保存他们所有的实例方法的真正所在.在JS中,`prototype`是不可枚举的,所以`for-in`无法发现.

每个函数包含两个非继承而来的方法`apply()`和`call()`.这两个方法都是在特定的作用域中调用函数,实际上是设置`this`的对象的值.不同的地方在于,`apply()`接收两个参数,一个是作用域对象,另一个是参数数组(可以是`Array`也可以是`arguments`).`call()`第一个参数是作用域对象,之后的参数直接传递给函数.

```js
//call
//function.call(thisArg, arg1, arg2, ...)
sayColor.call(null) //red
sayColor.call(window) //red
sayColor.call(o) //blue

//apply
//function.apply(thisArg, [argsArray])
sayColor.apply(null) //red
sayColor.apply(window) //red
sayColor.apply(o) //blue
```

还有一个方法:`bind()`.这个方法会创建一个函数实例.其`this`值会绑定到传给函数的值.

```js
//bind
var objSayColor = sayColor.bind(o);
//do some thing
console.log("======")
objSayColor() //blue
```
>注意:`bind()`返回的是函数,所以可以延迟调用,而`call()`和`apply()`是立即执行的.

 **参考**:

 - [apply](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
 - [call](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
 - [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)



