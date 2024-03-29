---
title: 浅谈JavaScript 的基本类型和引用类型
date: 2022-05-15 14:21:32
updated: 2022-06-09 16:08:33
tags: ['JavaScript']
categories:
 - JavaScript
---

# 浅谈JavaScript 的基本类型和引用类型

一个变量可以存放两种类型的值，基本类型的值（primitive values）和引用类型的值（reference values）。

ES6 引入了一种新的原始数据类型 **Symbol**，表示独一无二的值。它是 JavaScript 语言的第七种数据类型，前六种是：Undefined、Null、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object）。

<!--more-->

# 基本类型

JavaScript 中共有 6 种基本数据类型：`Undefined`、`Null`、`Boolean`、`Number`、`String`、`Symbol (new in ES 6)` ！

**约定：**`基本数据类型`与`原始数据类型`等意。

**基本数据类型的值是按值访问的。**

- 基本类型的值是不可变的

  ```javascript
  var str = "123hello321";
  str.toUpperCase();     // 123HELLO321
  console.log(str);      // 123hello321
  ```

- 基本类型的比较是它们的值的比较

  ```javascript
  var a = 1;
  var b = true;
  console.log(a == b);    // true
  console.log(a === b);   // false
  ```

  上面 a 和 b 的数据类型不同，但是也可以进行值的比较，这是因为在比较之前，自动进行了数据类型的 `隐式转换`。

  - `==` : 只进行值的比较
  - `===` : 不仅进行值得比较，还要进行数据类型的比较

- 基本类型的变量是存放在栈内存（Stack）里的

  ```javascript
  var a,b;
  a = "zyj";
  b = a;
  console.log(a);   // zyj
  console.log(b);   // zyj
  a = "呵呵";       // 改变 a 的值，并不影响 b 的值
  console.log(a);   // 呵呵
  console.log(b);   // zyj
  ```

图解如下：栈内存中包括了变量的标识符和变量的值

<img src="https://user-images.githubusercontent.com/33454514/172796362-5248f8fc-9339-4d7f-82f2-f7c6f85c5dc0.png" alt="image" style="zoom:67%;" />

# 引用类型

除过上面的 6 种基本数据类型外，剩下的就是引用类型了，统称为 `Object 类型`。细分的话，有：`Object 类型`、`Array 类型`、`Date 类型`、`RegExp 类型`、`Function 类型` 等。

## 引用类型的值按引用访问

- 引用类型的值是可变的

  ```javascript
  var obj = {name:"zyj"};   // 创建一个对象
  obj.name = "percy";       // 改变 name 属性的值
  obj.age = 21;             // 添加 age 属性
  obj.giveMeAll = function(){
    return this.name + " : " + this.age;
  };                        // 添加 giveMeAll 方法
  obj.giveMeAll();
  ```

- 引用类型的比较是引用的比较

  ```javascript
  var obj1 = {};    // 新建一个空对象 obj1
  var obj2 = {};    // 新建一个空对象 obj2
  console.log(obj1 == obj2);    // false
  console.log(obj1 === obj2);   // false
  ```

  因为 obj1 和 obj2 分别引用的是存放在堆内存中的2个不同的对象，故变量 obj1 和 obj2 的值（引用地址）也是不一样的！

- 引用类型的值是保存在堆内存（Heap）中的对象（Object）
  与其他编程语言不同，JavaScript 不能直接操作对象的内存空间（堆内存）。

  ```javascript
  var a = {name:"percy"};
  var b;
  b = a;
  a.name = "zyj";
  console.log(b.name);    // zyj
  b.age = 22;
  console.log(a.age);     // 22
  var c = {
    name: "zyj",
    age: 22
  };
  ```

图解如下：

- 栈内存中保存了变量标识符和指向堆内存中该对象的指针
- 堆内存中保存了对象的内容

<img src="https://user-images.githubusercontent.com/33454514/172796618-9f56c9c7-feb5-4ad4-8e46-a2a6652a51c1.png" alt="image" style="zoom:67%;" />

# 检测类型

- [typeof](https://link.segmentfault.com/?enc=DeveEaYaUUqHPJF1znucIw%3D%3D.ETqxYk%2BS3XEcR1223KFmADbKqjhwPJ1KJRSU4vHKVG7MRq8el2AP%2FnSZJVTnJi1rNSHWmJaqHJ3VbEanCf1c9FDc3sXX%2FAdxOIfhL13sgRALjBLwJ15jdc%2F7u1jBJYUT)：经常用来检测一个变量是不是最基本的数据类型

  ```javascript
  var a;
  typeof a;    // undefined
  
  a = null;
  typeof a;    // object
  
  a = true;
  typeof a;    // boolean
  
  a = 666;
  typeof a;    // number 
  
  a = "hello";
  typeof a;    // string
  
  a = Symbol();
  typeof a;    // symbol
  
  a = function(){}
  typeof a;    // function
  
  a = [];
  typeof a;    // object
  a = {};
  typeof a;    // object
  a = /aaa/g;
  typeof a;    // object   
  ```

- [instanceof](https://link.segmentfault.com/?enc=cZWjGieVJxR0sybfz0KPhA%3D%3D.4PMeTJK%2FoIycPv4gsMA5woH0i8NEr4EoWOIEtqbbaM7Jc0KB%2FFirqwz6cFxx51PP%2BzzmBG2L5mxdOgAWAM9KDZz%2Fil6ygtDgUEIhXixSWMDe6oUhwQmDHSL7wNMPa7dJ)：用来判断某个构造函数的 prototype 属性所指向的对象是否存在于另外一个要检测对象的原型链上

  - 简单说就是判断一个引用类型的变量具体是不是某种类型的对象

    ```javascript
    ({}) instanceof Object              // true
    ([]) instanceof Array               // true
    (/aa/g) instanceof RegExp           // true
    (function(){}) instanceof Function  // true
    ```
