---
title: co 函数库的含义和用法
date: 2022-05-25 14:21:32
updated: 2022-06-20 03:37:46
tags: ['Javascript']
categories:
- Javascript
---

# 什么是 co 函数库

[co 函数库](https://github.com/tj/co)是著名程序员 TJ Holowaychuk 于2013年6月发布的一个小工具，用于 Generator 函数的自动执行。

<!--more-->

比如，有一个 `Generator `函数，用于依次读取两个文件：

```js
var gen = function* (){
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

`co `函数库可以让你不用编写 `Generator `函数的执行器：

```js
var co = require('co');
co(gen);
```

上面代码中，`Generator `函数只要传入 `co `函数，就会自动执行。

`co` 函数返回一个 `Promise `对象，因此可以用 `then `方法添加回调函数。

```js
co(gen).then(function (){
  console.log('Generator 函数执行完成');
})
```

上面代码中，等到 `Generator `函数执行结束，就会输出一行提示。

# co 函数库的原理

为什么 `co` 可以自动执行 `Generator `函数？

前面文章说过，`Generator `函数就是一个异步操作的容器。它的自动执行需要一种机制，当异步操作有了结果，能够自动交回执行权。

两种方法可以做到这一点：

1. 回调函数。将异步操作包装成 `Thunk `函数，在回调函数里面交回执行权。

2. `Promise `对象。将异步操作包装成 `Promise `对象，用 `then `方法交回执行权。

`co `函数库其实就是将两种自动执行器（`Thunk `函数和 `Promise `对象），包装成一个库。使用 `co `的前提条件是，`Generator `函数的 `yield `命令后面，只能是 `Thunk `函数或 `Promise `对象。

# 基于 Promise 对象的自动执行

还是沿用上面的例子。首先，把` fs` 模块的 `readFile `方法包装成一个 `Promise `对象。

```js
var fs = require('fs');

var readFile = function (fileName){
  return new Promise(function (resolve, reject){
    fs.readFile(fileName, function(error, data){
      if (error) reject(error);
      resolve(data);
    });
  });
};

var gen = function* (){
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

然后，手动执行上面的 Generator 函数。

```js
var g = gen();

g.next().value.then(function(data){
  g.next(data).value.then(function(data){
    g.next(data);
  });
})
```

手动执行其实就是用 then 方法，层层添加回调函数。理解了这一点，就可以写出一个自动执行器。

```js
function run(gen){
  var g = gen();

  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(function(data){
      next(data);
    });
  }

  next();
}

run(gen);
```

上面代码中，只要 Generator 函数还没执行到最后一步，next 函数就调用自身，以此实现自动执行。

# co 函数库的源码

co 就是上面那个自动执行器的扩展，它的[源码](https://github.com/tj/co/blob/master/index.js)只有几十行，非常简单。

首先，co 函数接受 Generator 函数作为参数，返回一个 Promise 对象。

```js
function co(gen) {
  var ctx = this;

  return new Promise(function(resolve, reject) {
  });
}
```

在返回的 Promise 对象里面，co 先检查参数 gen 是否为 Generator 函数。如果是，就执行该函数，得到一个内部指针对象；如果不是就返回，并将 Promise 对象的状态改为 resolved 。

```js
function co(gen) {
  var ctx = this;

  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') gen = gen.call(ctx);
    if (!gen || typeof gen.next !== 'function') return resolve(gen);
  });
}
```

接着，co 将 Generator 函数的内部指针对象的 next 方法，包装成 onFulefilled 函数。这主要是为了能够捕捉抛出的错误。

```js
function co(gen) {
  var ctx = this;

  return new Promise(function(resolve, reject) {
    if (typeof gen === 'function') gen = gen.call(ctx);
    if (!gen || typeof gen.next !== 'function') return resolve(gen);

    onFulfilled();
    function onFulfilled(res) {
      var ret;
      try {
        ret = gen.next(res);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }    
  });
}
```

最后，就是关键的 next 函数，它会反复调用自身。

```js
function next(ret) {
  if (ret.done) return resolve(ret.value); // 检查当前是否为 Generator 函数的最后一步，如果是就返回
  var value = toPromise.call(ctx, ret.value); // 确保每一步的返回值是 Promise 对象
  if (value && isPromise(value)) return value.then(onFulfilled, onRejected); // 使用 then 方法，为返回值加上回调函数，然后通过 onFulfilled 函数再次调用 next 函数
  return onRejected(new TypeError('You may only yield a function, promise, generator, array, or object, '
        + 'but the following object was passed: "' + String(ret.value) + '"'));
    } // 在参数不符合要求的情况下（参数非 Thunk 函数和 Promise 对象），将 Promise 对象的状态改为 rejected，从而终止执行。
});
```

# 并发的异步操作

co 支持并发的异步操作，即允许某些操作同时进行，等到它们全部完成，才进行下一步。

这时，要把并发的操作都放在数组或对象里面。

```js
// 数组的写法
co(function* () {
  var res = yield [
    Promise.resolve(1),
    Promise.resolve(2)
  ];
  console.log(res); 
}).catch(onerror);

// 对象的写法
co(function* () {
  var res = yield {
    1: Promise.resolve(1),
    2: Promise.resolve(2),
  };
  console.log(res); 
}).catch(onerror);
```

