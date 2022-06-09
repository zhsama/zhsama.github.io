---
title: 浅谈 JavaScript 的模块规范
date: 2022-05-15 14:21:32
updated: 2022-06-09 16:08:33
tags: ['JavaScript']
categories:
 - JavaScript
---

# CommonJS

为了在浏览器外解决 JS 没有模块化和作用域问题，建立的一套模块规范，是 NodeJS 的模块系统采用的规范。

- 一个文件即一个模块
- 模块间作用域隔离
- 同步加载执行模块
- 通过 `module.exports`导出模块
- 通过 `require` 函数导入模块（导入的即 exports 导出的）

<!-- more -->

范例：

```js
// 定义模块math.js
var basicNum = 0;
function add(a, b) {
  return a + b;
}
module.exports = { //在这里写上需要向外暴露的函数、变量
  add: add,
  basicNum: basicNum
}

// 引用自定义的模块时，参数包含路径，可省略.js
var math = require('./math');
math.add(2, 5);

// 引用核心模块时，不需要带路径
var http = require('http');
http.createService(...).listen(3000);
```

CommonJS用同步的方式加载模块。在服务端，模块文件都存在本地磁盘，读取非常快，所以这样做不会有问题。但是在浏览器端，限于网络原因，更合理的方案是使用异步加载。

更多请查看 [CommonJS wiki](https://en.wikipedia.org/wiki/CommonJS)

# AMD规范

AMD 全称 `Asynchronous Module Definition`（异步模块定义），是为了实现 JS 在浏览器模块化所建立的规范。

NodeJS 基于文件系统，可以同步读取文件编译执行代码，但在浏览器中 JS 文件是通过网络请求加载，如果`require`时才加载会阻塞会后面代码的执行，因此 AMD 采用提前声明的方式异步导入模块。

主流实现是 [RequireJS](https://requirejs.org/)。

- 通过闭包隔离作用域
- 依赖前置，模块代码提前执行
- 通过 `define(id?, dependencies?, factory)` 函数定义模块
- 提前声明 `dependencies` 并分析依赖，在 `factory` 参数列表中取到导入的模块
- 通过 `return` 导出模块
- 设置 `define.amd` 对象用来判断当前是否是 AMD 规范模块加载器

范例：

```js
// m1.js
define('m1', function() {
  return {
    a: 1
  }
});
// m2.js
define('m2', function() {
  return {
    a: 2
  }
});

// entry.js
define('entry', ['m1', 'm2'], function(m1, m2) {
  console.log(m1.a + m2.a); // 输出 3
});
```

更多请查看 [AMD 规范](https://github.com/amdjs/amdjs-api/wiki/AMD)

# CMD规范

CMD 全称 Common Module Definition（普通模块定义），是国内开发者玉伯建立的模块化规范，初衷是让 CommonJS Modules/1.1 模块也能便捷快速迁移运行在浏览器端（NodeJS 的生态繁荣），因此更贴近 CommonJS 规范。

主流实现是 [SeaJS](https://seajs.github.io/seajs/docs/)

- 通过闭包隔离作用域
- 依赖就近，模块代码就近执行
- 通过 `define(factory(require, exports, module) {})` 函数定义模块
- 使用`require` 函数导入模块
- 使用 `exports` 导出模块
- 设置 `define.cmd` 对象用来判断当前是否是 CMD 规范模块加载器

范例：

```js
/** AMD写法 **/
define(["a", "b", "c", "d", "e", "f"], function(a, b, c, d, e, f) { 
     // 等于在最前面声明并初始化了要用到的所有模块
    a.doSomething();
    if (false) {
        // 即便没用到某个模块 b，但 b 还是提前执行了
        b.doSomething()
    } 
});

/** CMD写法 **/
define(function(require, exports, module) {
    var a = require('./a'); //在需要时申明
    a.doSomething();
    if (false) {
        var b = require('./b');
        b.doSomething();
    }
});

/** sea.js **/
// 定义模块 math.js
define(function(require, exports, module) {
    var $ = require('jquery.js');
    var add = function(a, b){
        return a + b;
    }
    exports.add = add;
});
// 加载模块
seajs.use(['math.js'], function(math){
    var sum = math.add(1 + 2);
});
```

更多请查看 [CMD 规范](https://github.com/seajs/seajs/issues/242)，[从 CommonJS 到 Sea.js](https://github.com/seajs/seajs/issues/269)

# UMD

UMD 全称 Universal Module Definition（通用模块定义），UMD 的诞生是为了兼容 CommonJS，AMD，CMD 多种规范的一种模块实现，保证模块多端可运行，所以严格来说 UMD 并不算一种模块规范。

UMD 模块的实现：

```js
(function (global, factory) {
  if (typeof module === 'object' && module.exports) {
    module.exports = factory();
  } else if (typeof define === 'function' && define.amd) {
    define(factory);
  } else if (typeof define === 'function' && define.cmd) {
    define(function(require, exports, module) {
      module.exports = factory();
    });
  } else {
    global.returnExports = factory();
  }
})(this, function() {
  // 模块代码
});
```

# ESM

ESM 全称 ES Module，是 ES6 提出的模块解决方案

- 编译时加载
- 使用`import`关键字导入模块
- 使用`export`关键字导出模块

范例：

```js
/** 定义模块 math.js **/
var basicNum = 0;
var add = function (a, b) {
    return a + b;
};
export { basicNum, add };

/** 引用模块 **/
import { basicNum, add } from './math';
function test(ele) {
    ele.textContent = add(99 + basicNum);
}

```

如上例所示，使用`import`命令的时候，用户需要知道所要加载的变量名或函数名。其实ES6还提供了`export default`命令，为模块指定默认输出，对应的`import`语句不需要使用大括号。这也更趋近于AMD的引用写法。

```js
/** export default **/
//定义输出
export default { basicNum, add };

//引入
import math from './math';
function test(ele) {
  ele.textContent = math.add(99 + math.basicNum);
}
```
