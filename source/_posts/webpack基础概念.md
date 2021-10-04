---
title: webpack基础及易混淆的概念
date: 2021-10-05 00:16:03
updated: 2021-10-05 02:25:45
tags: ['前端','webpack', '八股文']
categories:
 - webpack
 - 八股文
 - 前端
description:
---
Webpack 有非常多的概念，很多名词长得都差不多。本文旨在将这些分散在文档和教程里的内容总结起来，完成一份 webpack 中的易混淆知识点集合。
    
<!-- more -->
# module，chunk 和 bundle 的区别
## 概念说明
webpack 官网对 chunk 和 bundle 的解释过于抽象。为方便理解，我这里举个例子给大家形象化的解释一下。

我们在 src 目录下引入 index.js、utils.js、common.js 和 index.css 这 4 个文件，目录结构如下：
```
src/
├── index.css
├── index.html # HTML 模板代码
├── index.js
├── common.js
└── utils.js
```
index.css 的内容：
```css
body {
    background-color: red;
}
```
utils.js 的内容：
```js
export function Fibonacci(n) {
  if (n <= 2) {
    return n
  }
  let n1 = 1, n2 = 1, sum;
  for (let i = 2; i < n; i++) {
    sum = n1 + n2
    n1 = n2
    n2 = sum
  }
  return sum
}
```
common.js 的内容：
```js
export default {
  log: (msg) => {
    console.log('hello ', msg)
  }
}
```
index.js 的内容：
```js
import './index.css';
import Common from './common.js'

Common.log('webpack');
```
webpack 的配置如下:
```js
{
    entry: {
        index: "../src/index.js",
        utils: '../src/utils.js',
    },
    output: {
        filename: "[name].bundle.js", // 输出 index.js 和 utils.js
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    MiniCssExtractPlugin.loader, // 创建一个 link 标签
                    'css-loader', // css-loader 负责解析 CSS 代码, 处理 CSS 中的依赖
                ],
            },
        ]
    }
    plugins: [
        // 用 MiniCssExtractPlugin 抽离出 css 文件，以 link 标签的形式引入样式文件
        new MiniCssExtractPlugin({
            filename: 'index.bundle.css' // 输出的 css 文件名为 index.css
        }),
    ]
}
```
运行 webpack后包的结果：
![image](https://user-images.githubusercontent.com/33454514/135892008-9f99b7f4-582c-4089-9107-aced8feb8621.png)

由上图可以看出，`index.css` 和 `common.js` 在 `index.js` 中被引入，打包生成的 `index.bundle.css` 和 `index.bundle.js` 都属于 `chunk 0`，`utils.js` 因为是独立打包的，它生成的 `utils.bundle.js` 属于 `chunk 1`。

![image](https://user-images.githubusercontent.com/33454514/135892433-d8191708-0dc5-4b78-84aa-057fbdf985b2.png)

看这个图就很明白了：

1. 对于一份同逻辑的代码，当我们手写下一个一个的文件，它们无论是 ESM 还是 commonJS 或是 AMD，他们都是 module ；
2. 当我们写的 module 源文件传到 webpack 进行打包时，webpack 会根据文件引用关系生成 chunk 文件，webpack 会对这个 chunk 文件进行一些操作；
3. webpack 处理好 chunk 文件后，最后会输出 bundle 文件，这个 bundle 文件包含了经过加载和编译的最终源文件，所以它可以直接在浏览器中运行。
一般来说一个 chunk 对应一个 bundle，比如上图中的 utils.js -> chunks 1 -> utils.bundle.js；但也有例外，比如说上图中，我就用 MiniCssExtractPlugin 从 chunks 0 中抽离出了 index.bundle.css 文件。
## 总结
`module`，`chunk` 和 `bundle` 其实就是同一份逻辑代码在不同转换场景下的取了三个名字：

我们直接写出来的是 `module`，`webpack` 处理时是 `chunk`，最后生成浏览器可以直接运行的 `bundle`。
# filename 和 chunkFilename 的区别
## filename
`filename` 是一个很常见的配置，就是对应于 `entry` 里面的输入文件，`经过webpack` 打包后输出文件的文件名。比如说经过下面的配置，生成出来的文件名为 `index.min.js`。
```js
{
    entry: {
        index: "../src/index.js"
    },
    output: {
        filename: "[name].min.js", // index.min.js
    }
}
```

![image](https://user-images.githubusercontent.com/33454514/135893374-3f5620a6-e9d5-46b6-b65a-0c4afbb3ebb1.png)
## chunkFilename
`chunkFilename` 指未被列在 `entry` 中，却又需要被打包出来的 `chunk` 文件的名称。一般来说，这个 `chunk` 文件指的就是要懒加载的代码。

比如在代码中写了一份懒加载 lodash 的代码：
```js
// 文件：index.js

// 创建一个 button
let btn = document.createElement("button");
btn.innerHTML = "click me";
document.body.appendChild(btn);

// 异步加载代码
async function getAsyncComponent() {
  var element = document.createElement('div');
  const { default: _ } = await import('lodash');
  element.innerHTML = _.join(['Hello!', 'dynamic', 'imports', 'async'], ' ');

  return element;
}

// 点击 button 时，懒加载 lodash，在网页上显示 Hello! dynamic imports async
btn.addEventListener('click', () => {
  getAsyncComponent().then(component => {
    document.body.appendChild(component);
  })
})
```

`webpack` 配置不做任何变动，这时候的打包结果如下：

![image](https://user-images.githubusercontent.com/33454514/135893745-8d0773ff-65dd-446d-ad7f-eccf25bc76e9.png)
这个 `1.min.js` 就是异步加载的 `chunk` 文件。官方文档的解释如下：

> `output.chunkFilename` 默认使用 `[id].js` 或从 `output.filename` 中推断出的值（`[name]` 会被预先替换为 `[id]` 或 `[id].`）

结合上面的例子来看：

`output.filename` 的输出文件名是 `[name].min.js`，`[name]` 根据 `entry` 的配置推断为 `index`，所以输出为 `index.min.js`；

由于 `output.chunkFilename` 没有显示指定，就会把 `[name]` 替换为 `chunk` 文件的 `id` 号，这里文件的 `id` 号是 1，所以文件名就是 `1.min.js`。

如果显式地配置 `chunkFilename`，就会按配置的名字生成文件：

```js
{
  entry: {
    index: "../src/index.js"
  },
  output: {
    filename: "[name].min.js",  // index.min.js
    chunkFilename: 'bundle.js', // bundle.js
  }
}
```

打包的结果如下图：

![image](https://user-images.githubusercontent.com/33454514/135894219-b65bd09a-b2cf-434c-9847-ecc5821ca342.png)

## 总结

`filename` 指列在 `entry` 中，打包后输出的文件的名称。

`chunkFilename` 指未列在 `entry` 中，却又需要被打包出来的文件的名称。

# webpackPrefetch、webpackPreload 和 webpackChunkName 的区别

前面举了个异步加载 `lodash` 的例子，我们最后把 `output.chunkFilename` 写死成 `bundle.js`。在我们的业务代码中，不可能只异步加载一个文件，所以写死肯定是不行的，但是写成 `[name].bundle.js` 时，打包的文件又是意义不明、辨识度不高的 `chunk id`。

这时候 `webpackChunkName` 就可以派上用场了。我们可以在 `import` 文件时，在 `import` 里以注释的形式为 `chunk` 文件取别名：

```js
async function getAsyncComponent() {
  var element = document.createElement('div');
  
  // 在 import 的括号里 加注释 /* webpackChunkName: "lodash" */ ，为引入的文件取别名
  const { default: _ } = await import(/* webpackChunkName: "lodash" */ 'lodash');
  element.innerHTML = _.join(['Hello!', 'dynamic', 'imports', 'async'], ' ');

  return element;
}
```

这时候打包生成的文件是这样的：

![image](https://user-images.githubusercontent.com/33454514/135894906-16f58da4-5910-41bb-933d-b58632b5d5f8.png)

现在问题来了，`lodash` 是我们取的名字，按道理来说应该生成 `lodash.bundle.js` 啊，前面的 `vendors~` 是什么玩意？

其实 `webpack` 懒加载是用内置的一个插件 [SplitChunksPlugin](https://webpack.docschina.org/plugins/split-chunks-plugin/) 实现的，这个插件里面有些默认 [配置项](https://webpack.docschina.org/plugins/split-chunks-plugin/#optimization-splitchunks) ，比如说 `automaticNameDelimiter`，默认的分割符就是 `~`，所以最后的文件名才会出现这个符号。

## webpackPrefetch 和 webpackPreload

这两个配置一个叫预拉取（Preload），一个叫预加载（Prefetch），两者有些细微的不同，我们先说说 `webpackPreload`。

在上面的懒加载代码里，我们是点击按钮时，才会触发异步加载 lodash 的动作，这时候会动态的生成一个 `script` 标签，加载到 `head` 头里：


![image](https://user-images.githubusercontent.com/33454514/135895496-b3d2e1b7-1a82-4f92-b1a6-946552410ac3.png)

如果我们 `import` 的时候添加 `webpackPrefetch`：

```js
const { default: _ } = await import(/* webpackChunkName: "lodash" */ /* webpackPrefetch: true */ 'lodash');
```

就会以 <link rel="prefetch" as="script"> 的形式预拉取 lodash 代码：

![image](https://user-images.githubusercontent.com/33454514/135900665-8bbacb1c-2250-4e35-8807-19dc3ad59f60.png)

这个异步加载的代码不需要手动点击 button 触发，webpack 会在父 chunk 完成加载后，闲时加载 lodash 文件。

webpackPreload 是预加载当前导航下可能需要资源，他和 webpackPrefetch 的主要区别是：

* `preload chunk` 会在父 `chunk` 加载时，以并行方式开始加载。`prefetch chunk` 会在父 `chunk` 加载结束后开始加载。
* `preload chunk` 具有中等优先级，并立即下载。`prefetch chunk` 在浏览器闲置时下载。
* `preload chunk` 会在父 `chunk` 中立即请求，用于当下时刻。`prefetch chunk`会用于未来的某个时刻

## 总结

`webpackChunkName` 是为预加载的文件取别名，`webpackPrefetch` 会在浏览器闲置下载文件，`webpackPreload` 会在父 `chunk` 加载时并行下载文件。

# hash、chunkhash、contenthash 的区别

## hash

`hash` 计算是跟整个项目的构建相关，沿用例一中的项目路径。

修改`webpack` 的核心配置如下:
```js
{
    entry: {
        index: "../src/index.js",
        utils: '../src/utils.js',
    },
    output: {
        filename: "[name].[hash].js",  // 改为 hash
    },
    
    ......
    
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'index.[hash].css' // 改为 hash
        }),
    ]
}
```

打包后生成的文件名如下：

![image](https://user-images.githubusercontent.com/33454514/135901307-475f7263-9f07-41ec-9dcc-6a883be5d5d5.png)

不难发现：生成文件的 `hash` 和项目的构建 `hash` 都是一模一样的。

## chunkhash

因为 `hash` 是项目构建的哈希值，项目中如果有些变动，`hash` 一定会变，比如修改 `utils.js` 的代码，`index.js` 里的代码虽然没有改变，但是大家都是用的同一份 `hash`。`hash` 一变，缓存一定失效了，这样子是没办法实现 CDN 和浏览器缓存的。

`chunkhash` 就是解决这个问题的，它根据不同的入口文件(Entry)进行依赖文件解析、构建对应的 `chunk`，生成对应的哈希值。

例如对 `utils.js` 里文件进行改动：

```js
function Fibonacci(n) {
  if (n <= 2) {
    return n
  }
  let n1 = 1, n2 = 1, sum;
  for (let i = 2; i < n; i++) {
    sum = n1 + n2
    n1 = n2
    n2 = sum
  }
  return sum
}

function cube(x) {
  return x * x * x;
}


const utils = {
  cube,
  Fibonacci
}
export default utils;
```

然后把 `webpack` 里的所有 `hash` 改为 `chunkhash`：
```js
{
    entry: {
        index: "../src/index.js",
        utils: '../src/utils.js',
    },
    output: {
        filename: "[name].[chunkhash].js", // 改为 chunkhash
    },
          
    ......
    
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'index.[chunkhash].css' // // 改为 chunkhash
        }),
    ]
}
```

![image](https://user-images.githubusercontent.com/33454514/135901840-cbb01f5f-dec9-4d17-b860-d7bedf02975e.png)

我们可以看出，`chunk 0` 的 `hash` 都是一样的，`chunk 1` 的 `hash` 和上面的不一样。

假设我又把 `utils.js` 里的 `cube()` 函数去掉，再打包：

![image](https://user-images.githubusercontent.com/33454514/135901951-4ed6daed-263a-44eb-a6e0-8f78cd5fda2f.png)

对比可以发现，只有 chunk 1 的 hash 发生变化，chunk 0 的 hash 还是原来的。

## contenthash

我们更近一步，`index.js` 和 `index.css` 同为一个 `chunk`，如果 `index.js` 内容发生变化，但是 `index.css` 没有变化，打包后他们的 `hash` 都发生变化，这对 `css` 文件来说是一种浪费。如何解决这个问题呢？

`contenthash` 将根据资源内容创建出唯一 `hash`，也就是说文件内容不变，`hash` 就不变。

修改 webpack 的配置如下：

```js
{
    entry: {
        index: "../src/index.js",
        utils: '../src/utils.js',
    },
    output: {
        filename: "[name].[chunkhash].js",
    },
      
    ......
    
    plugins: [
        new MiniCssExtractPlugin({
            filename: 'index.[contenthash].css' // 这里改为 contenthash
        }),
    ]
}
```

在对 index.js 文件进行三次修改，打包的结果分别如下图：

![image](https://user-images.githubusercontent.com/33454514/135902268-f0b0b2bd-2efb-4f18-bb5d-2603b375720f.png)

![image](https://user-images.githubusercontent.com/33454514/135902274-7afa84b0-5d2f-4654-a9ba-12899bc2bc8e.png)

![image](https://user-images.githubusercontent.com/33454514/135902277-535cc6c3-e5cc-4e3b-89d4-e7b3ed1781f5.png)

不难看出，`css` 文件的 `hash` 始终没有发生改变。

## 总结 

`hash` 计算与整个项目的构建相关；

`chunkhash` 计算与同一 `chunk` 内容相关；

`contenthash` 计算与文件内容本身相关。

# sourse-map 中 eval、cheap、inline 和 module 的区别

简单说，Source map就是一个信息文件，里面储存着位置信息。也就是说，转换后的代码的每一个位置，所对应的转换前的位置。

有了它，出错的时候，除错工具将直接显示原始代码，而不是转换后的代码。这无疑给开发者带来了很大方便。

官网上的sourcemap种类如下图：
![image](https://user-images.githubusercontent.com/33454514/135902685-0e8e17f7-4d43-401a-9fa9-eca8961ba0f4.png)

观察后不难发现大部分都是 eval、cheap、inline 和 module这 4 个词排列组合的，于是可以大致归纳为如下四个种类：

|  参数   | 表头  |
|  ----  | ----  |
| eval  | 打包后的模块都使用 `eval()` 执行，行映射可能不准；不产生独立的 `map` 文件 |
| cheap  | map 映射只显示行不显示列，忽略源自 `loader` 的 `source-map`|
| inline  | 映射文件以 `base64` 格式编码，加在 `bundle` 文件最后，不产生独立的 `map` 文件|
| module  | 增加对 `loader` `source` `map` 和第三方模块的映射
|

## source-map

source-map 是最大而全的，会生成独立 map 文件

![image](https://user-images.githubusercontent.com/33454514/135903295-45c87381-799b-45aa-a79d-460c76e61c40.png)

source-map 会显示报错的行列信息：

![image](https://user-images.githubusercontent.com/33454514/135903373-dec99574-7ea9-4493-8a8b-e36a7f42a59e.png)

## cheap-sourse-map

`cheap` 不会产生列映射，相应的体积会小很多，和 `sourse-map` 的打包结果比一下发现体积只有原来的 1/4 。

![image](https://user-images.githubusercontent.com/33454514/135903497-64abffc3-551d-41c1-8897-15a531ba0bd9.png)

## eval-source-map

`eval-source-map` 会以 `eval()` 函数打包运行模块，不产生独立的 `map` 文件，会显示报错的行列信息：

![image](https://user-images.githubusercontent.com/33454514/135903556-9a39d9ad-be3d-469c-8dfc-5809b941593a.png)

![image](https://user-images.githubusercontent.com/33454514/135903552-00a05907-f6d7-4362-9a96-4039a45be86f.png)

```js
// index.bundle.js 文件

!function(e) {
    // ......
    // ......
    // ......
}([function(module, exports) {
    eval("console.lg('hello source-map !');" +
      "//# sourceURL=[module]\n//# " +
      "sourceMappingURL=data:application/json;" +
      "charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJz" +
      "b3VyY2VzIjpbIndlYnBhY2s6Ly8vLi4vc3JjL2luZG" +
      "V4Mi5qcz9mNmJjIl0sIm5hbWVzIjpbImNvbnNvbGUiL" +
      "CJsZyJdLCJtYXBwaW5ncyI6IkFBQUFBLE9BQU8sQ0F" +
      "BQ0MsRUFBUixDQUFXLG9CQUFYIiwiZmlsZSI6IjAua" +
      "nMiLCJzb3VyY2VzQ29udGVudCI6WyJjb25zb2xlLmx" +
      "nKCdoZWxsbyBzb3VyY2UtbWFwICEnKSJdLCJzb3VyY2" +
      "VSb290IjoiIn0=\n//" +
      "# sourceURL=webpack-internal:///0\n")
}
]);
```

## inline-source-map

映射文件以 `base64` 格式编码，加在 `bundle` 文件最后，不产生独立的 `map` 文件。

加入 `map` 文件后可以明显的看到包体积变大了：

```js
// index.bundle.js 文件

!function(e) {

}([function(e, t) {
    console.lg("hello source-map !")
}
]);

// # sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJzb3Vy...
// base64编码
```

## 常用配置

1. source-map

  * 大而全，啥都有，会让 webpack 构建时间变长，看情况使用。

2. cheap-module-eval-source-map 

  * 这个一般是开发环境（dev）推荐使用，在构建速度报错提醒上做了比较好的均衡。

3. cheap-module-source-map

  * 一般来说，生产环境是不配 `source-map` 的，如果想捕捉线上的代码报错，我们可以用这个
