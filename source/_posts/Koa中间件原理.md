---
title: Koa中间件原理 
date: 2022-02-28 01:44:24
updated: 2022-03-06 23:35:39
tags: [Koa, NodeJS]
categories: 
    - Koa
    - NodeJS
---

# 什么是中间件

中间件是介于应用系统和系统软件之间的一类软件，它使用系统软件所提供的基础服务（功能），衔接网络上应用系统的各个部分或不同的应用，能够达到资源共享、功能共享的目的。

在 `Nodejs` 中，**中间件**主要是指封装 **http** 请求细节处理的方法。我们都知道在 **http** 请求中往往会涉及很多动作，如下:

- IP 筛选
- 查询字符串传递
- 请求体解析
- cookie 信息处理
- 权限校验
- 日志记录
- 会话管理中间件(session)
- gzip 压缩中间件(如 compress)
- 错误处理

当然还有很多自定义的处理动作. 对于 **Web** 应用而言，我们并不希望了解每一个细节性的处理工作，而是希望能够把主要精力集中在业务的开发上，以达到提升开发效率的目的，所以引入了 **Node 中间件**
来简化和封装这些基础逻辑处理细节.

**node 中间件**本质上就是在进入具体的业务处理之前，先让**特定过滤器**处理。如下图所示:

![image](https://user-images.githubusercontent.com/33454514/156928218-c20afb06-df25-4dd1-b7d4-ff9a03a07084.png)

我们目前看到的主流  `Nodejs` 框架，比如 `koa，express，egg` 等，都离不开中间件的设计概念，所以为了能让大家更深入的窥探  `Nodejs` 世界，我们就非常有比较研究中间件的实现原理。

# node 中间件核心原理实现

由上文介绍可知，中间件是从 http 请求开始到响应结束过程中的处理逻辑，通常需要对请求和响应进行处理。我们在实现 node
中间件模式时还需要考虑的一个问题就是多中间件共存的问题，我们要思考如何将多个中间件的执行自动化，不然在请求到响应的过程中只会执行最开始的中间件，所以我们基本的中间件形式如下：

```js
const middleware = (req, res, next) => {  // 请求处理逻辑 
  next();
} 
```

接下来我们先写个简单的案例来看看中间件是如何实现的.

```js
// 定义几个中间间函数
const m1 = (req, res, next) => {
  console.log('m1 run')
  next()
}
const m2 = (req, res, next) => {
  console.log('m2 run')
  next()
}
const m3 = (req, res, next) => {
  console.log('m3 run')
  next()
}

// 中间件集合const middlewares = [m1, m2, m3]
function useApp(req, res) {
  const next = () => {    // 获取第一个中间件
    const middleware = middlewares.shift();
    if (middleware) {
      middleware(req, res, next);
    }
  }
  next();
}

// 第一次请求流进入
useApp();
```

由以上代码我们就不难发现 **next** 的作用了，也就是实现自动调用中间件链的关键参数. 打印结果如下:

```
m1 run
m2 runm3
run 
```

以上即实现了基本中间件的执行模式，但是我们还需要考虑异步的问题，如果中间件还依赖第三发模块或者 api 的支持，比如验证，识别等服务，我们需要在该异步中间件的回调里执行 next，才能保证正常的调用执行顺序，如下代码所示:

```js
const m2 = (req, res, next) => {
  fetch('/xxxxx').then(res => {
    next();
  });
} 
```

还有一种中间件场景，比如说日志中间件，请求监控中间件，它们会在业务处理前和处理后都会执行相关逻辑，这个时候就要求我们需要能对 **next** 函数进行二次处理，我们可以将 next 的返回值包装成 **promise**
，使得其在业务处理完成之后通过 **then** 回调来继续处理中间件逻辑. 如下所示:

```js
function useApp(req, res) {
  const next = () => {
    const middleware = middlewares.shift();
    if (middleware) {      // 将返回值包装为Promise对象
      return Promise.resolve(middleware(req, res, next));
    } else {
      return Promise.resolve("end");
    }
  }
  next();
} 
```

此时我们就能使用如下方式调用了:

```js
const m1 = (req, res, next) => {
  console.log('m1 start');
  return next().then(() => {
    console.log('m1 end');
  });
} 
```

以上我们就实现了一个基本可以的中间件设计模式，当然我们也可以用 async 和 await 实现，写法会更优雅和简单：

```js
const m1 = async (req, res, next) => {
  // something...
  let result = await next();
}
const m2 = async (req, res, next) => {
  // something... 
  let result = await next();
}
const m3 = async (req, res, next) => {
  // something... 
  let result = await next();
  return result;
}

const middlewares = [m1, m2, m3]
;

function useApp(req, res) {
  const next = () => {
    const middleware = middlewares.shift();
    if (middleware) {
      return Promise.resolve(middleware(req, res, next));
    } else {
      return Promise.resolve("end");
    }
  }
  next();
}

// 启动中间件useApp() 
```

在 koa2 框架中，中间件的实现方式也是将 `next()` 方法返回值封装为 Promise 对象，实现了其提出的洋葱圈模型：

![image](https://user-images.githubusercontent.com/33454514/156928188-db25c072-6697-49eb-a099-d940a69dd147.png)

# koa 中间件实现方式

koa2 框架的中间件实现原理很优雅，这里贴上源码：

```js
function compose(middleware) {  // 提前判断中间件类型,防止后续错误
  if (!Array.isArray(middleware)) {
    throw new TypeError('Middleware stack must be an array!')
  }
  for (const fn of middleware) {    // 中间件必须为函数类型
    if (typeof fn !== 'function') {
      throw new TypeError('Middleware must be composed of functions!')
    }
  }
  return function (context, next) {    // 采用闭包将索引缓存,来实现调用计数
    let index = -1;
    return dispatch(0);

    function dispatch(i) {      // 防止next()方法重复调用  
      if (i <= index) {
        return Promise.reject(new Error('next() called multiple times'))
      }
      index = i;
      let fn = middleware[i];
      if (i === middleware.length) {
        fn = next;
      }
      if (!fn) {
        return Promise.resolve();
      }
      try {        // 包装next()返回值为Promise对象  
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)));
      } catch (err) {        // 异常处理 
        return Promise.reject(err);
      }
    }
  }
}
```

# 实现一个自己的 koa 中间件

在服务端中，我们经常需要对用户登录权限进行过滤，此时我们提供统一个中间件来处理用户权限：

```js
// 模拟数据库操作
const token = db.user();

// router或者koa的中间件一定要用await处理next，否则将不能正常响应数据
export default async (ctx, next) => {
  const t = ctx.request.header.authorization;
  let uid = ctx.request.header['x-requested-with'];
  let uidArr = uid.split(',');
  if (uidArr.length > 1) {
    uid = uidArr.pop().trim()
  }
  if (token[uid] && token[uid][1] === t) {
    await next();
  } else {
    ctx.status = 403;
    ctx.body = {
      state: 403,
      msg: '未登录或没有权限'
    }
  }
} 
```

以上代码即实现用户登录态处理，如果用户在没有登录的情况下防问任何需要登录的接口，都将返回权限不足或则在请求库中让其重定向到登录页面
