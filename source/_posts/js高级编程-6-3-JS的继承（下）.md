---
title: JS的继承（下）
date: 2018-06-05 17:03:26
tags: js高级编程
---

继承是OO语言的一个重要概念。许多OO语言支持两种方式继承：接口继承和实现继承。接口继承只继承方法签名，而实现继承则继承实际的方法。ECMAScript函数没有签名，所以无法实现接口继承，只支持实现继承，实现继承主要依靠原型链来实现的。

<!-- more -->

### 原型链

原型链作为实现继承的主要方法，其基本思想是利用原型让每个引用类型继承另一个引用类型的属性和方法。


实现原型链有一种基本模式，其代码如下：
```js
 //super
 function SuperType() {
     this.property = true;
 }
 SuperType.prototype.getSuperValue = function () {
     return this.property;
 };

 //sub
 function SubType() {
     this.subProperty = false;
 }

 //继承SuperType
 SubType.prototype = new SuperType();

 SubType.prototype.getSubValue = function () {
     return this.subProperty;
 };

 let instance = new SubType();
 console.log("property : " + instance.getSuperValue()) //true
```
在上面的例子中，我们没有使用`SubType`的原型，而是给了它`SuperType`的实例。于是，新原型有了`SuperType`的所有属性和方法。通过实现原型链，实际上是扩展了原型搜索机制。

#### 1. 别忘记默认的原型
前面的例子少了一环。`SuperType`其实还包含了一个内部指针，指向`Object.prototype`。一句话，SubType继承了`SuperType`，`SuperType`继承了`Object`。
 
#### 2. 确定实例和原型的关系
可以通过两种方式确定原型和实例的关系。第一种是通过`instaceof`操作符。另一种是`isPropertyOf()`方法。

```js

 console.log(instance1 instanceof Object)     //true
 console.log(instance1 instanceof SuperType)   //true
 console.log(instance1 instanceof SubType)   //true

 console.log(Object.prototype.isPrototypeOf(instance1))   //true
 console.log(SuperType.prototype.isPrototypeOf(instance1))   //true
 console.log(SubType.prototype.isPrototypeOf(instance1))   //true

```

#### 3. 谨慎地定义方法
子类型有时候需要的重写超类型的某个方法，或者需要添加不存在的方法。添加方法的代码一定要放在替换原型之后！

```js
 //super
  function SuperType() {
      this.property = true;
  }
  SuperType.prototype.getSuperValue = function () {
      return this.property;
  };

  //sub
  function SubType() {
      this.subProperty = false;
  }

  //继承SuperType
  SubType.prototype = new SuperType();

  SubType.prototype.getSubValue = function () {
      return this.subProperty;
  };

  let instance = new SubType();
  console.log("property : " + instance.getSuperValue()) //true

  SubType.prototype.getSuperValue = function () {
      return false;
  };
  instance = new SubType();
  console.log("property : " + instance.getSuperValue()) //false
```
上述代码重写了`SubType`的`getSuperValue()`方法，当`SubType`实例调用`getSuperValue()`方法时，调用的是新的方法；但通过`SuperType`的实例调用`getSuperValue()`时，会继续调用原来的方法。

> 必须在实例替换原型之后，再定义方法。
> 通过原型链实现继承时，不能使用字面量创建原型方法，因为这样会重写原型链。

```js
  //super
  function SuperType() {
      this.property = true;
  }
  SuperType.prototype.getSuperValue = function () {
      return this.property;
  };

  //sub
  function SubType() {
      this.subProperty = false;
  }

  //继承SuperType
  SubType.prototype = new SuperType();

  SubType.prototype.getSubValue = function () {
      return this.subProperty;
  };
  
  SubType.prototype = {
      //变成了Obejct实例
      getSubValue: function () {
          return this.subProperty;
      },
      someMethod: function () {
          return false;
      }
  };
  
  let instance = new SubType();
  console.log("property : " + instance.getSuperValue()) //error

```

#### 4. 原型链的问题
原型链虽然很强大，可以用来实现继承，但也存在一些问题。最主要的问题是来自包含引用类型值的原型。引用类型的原型属性会被所有实例共享；这也是为什么要在构造函数而不是原型对象中定义属性的原因。

```js
function SuperType(){
    this.colors = ["red","blue","green"];
}

function SubType(){
}

//继承
SubType.prototype = new SuperType();

let ins1 = new SubType()
ins1.colors.push("black")
console.log(ins1.colors) //red , blue ,green ,black

let ins2 = new SubType()
console.log(ins2.colors)//red , blue ,green ,black
```
原型链的另一个问题是：创建子类型的实例时，无法向超类型的构造函数中传递参数。实际上是无法在不影响所有实例对象的情况下，给超类型传递参数。所以实践中很少单独使用原型链继承模式。











