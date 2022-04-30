---
title: 浅谈TS的接口和抽象类
date: 2022-03-25 14:21:32
updated: 2022-04-30 17:31:21
tags: ['typescript']
categories:
- typescript
---

接口和抽象类的出现主要是为了添加参数的限制，来规范代码。
<!--more-->

# 接口

TypeScript的核心原则之一是对值所具有的结构进行类型检查。 它有时被称做“鸭式辨型法”或“结构性子类型化”。 在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。 

关键词`interface` 和抽象类有一定的相似之处：

```ts
interface Eg {
	a: number; // 必须参数
	b?: string; // 可选参数
    readonly c?: string; // 只读的可选参数
    [prop:number]:boolean // 意思是索引值的类型要为number类型，值为布尔值
}
var data: Eg = { a: 1 ,b:"2",c:"3",1:true}
console.log(data)
```

## 接口中规定函数

```ts
interface Eg {
    (x:string,y:string): string // 规定函数
}
let eg: Eg;
eg = function (x:string,y:string):string {
    return "eg"
}
```

## 接口继承接口

关键字：`extends`

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}
let square = <Square>{}; 
let square:Square = {} // error 
```

上述两者意思都是使用Square这个接口，区别是一个需要声明后立即赋值，另一个不需要。

# 抽象类

为了规定类中的一些属性和方法，写出来就是为了被继承的，也只能被继承。

抽象类无法实列化，抽象类中也可以拥有一些抽象的属性和方法，和接口的主要区别是接口中无法实现方法，而抽象类的可以。 **关键字**`abstract`

```ts
abstract class Human {
    abstract move(x: number): any;
    constructor (readonly name:string) {}
    eat() {
         
     }
}
class Student extends Human {
    move(x:number) {
        // 从抽象类中继承来的抽象方法必须在派生类中实现
    }
    constructor (name:string) {
        super(name)
    }
}

abstract class Human<T> {
    abstract move(x: number):T;
    constructor (readonly name:string) {}
    eat() {
         
     }
}

```
## 接口继承类：
```ts
class Man {
    job: string;
    constructor (readonly name: string) {}
    earnMoney () {
        console.log(`earning money`)
    }
}
 
interface HumanRule extends Man{
    nose: string;
    mouth: string;
    ear: string;
    eye: string;
    eyebrow: string
}
```
当我们要使用这个接口的时候也要继承Man否则就是确实一个name属性，类继承接口使用关键字`implements`。
```ts
class a extends Man implements HumanRule{
    nose: string;
    mouth: string;
    ear: string;
    eye: string;
    eyebrow: string;
    constructor (name) {
    	super(name)
    }
}
```
`implements`也可以实现类的多继承。意思是当一个接口继承了类的属性或者方法之后，使用这个接口是也要继承那个类。

## 混合类型

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number) { };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}
let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```
