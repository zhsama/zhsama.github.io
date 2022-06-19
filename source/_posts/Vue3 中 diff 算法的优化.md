---
title: Vue3 中 diff 算法的优化
date: 2022-06-11 14:21:32
updated: 2022-06-20 03:39:28
tags: ['vue3']
categories:
- vue3
---

# Vue3 中 diff 算法的优化

在 Vue3 里整个重写了 Diff 算法。虽然源码的话可以说基本是完全不一样的，但是流程还是一样的。

关于 Vue3 的 Diff 完整源码解析还在撰稿中，过几天就发布了，这里先介绍一下相比 Vue2 优化的部分，尤大公布的数据就是 `update` 性能提升了 `1.3~2 倍`，`ssr` 性能提升了 `2~3 倍`，来看看都有哪些优化：

- 事件缓存：将事件缓存，可以理解为变成静态的了
- 添加静态标记：Vue2 是全量 Diff，Vue3 是静态标记 + 非全量 Diff
- 静态提升：创建静态节点时保存，后续直接复用
- 使用最长递增子序列优化了对比流程：Vue2 里在 updateChildren() 函数里对比变更，在 Vue3 里这一块的逻辑主要在 patchKeyedChildren() 函数里

<!--more-->


# 事件缓存

比如这样一个有点击事件的按钮：

```js
<button @click="handleClick">按钮</button>
```

在 Vue3 被编译后的结果：

```js
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("button", {
    onClick: _cache[0] || (_cache[0] = (...args) => (_ctx.handleClick && _ctx.handleClick(...args)))
  }, "按钮"))
}
```

`onClick `会先读取缓存，如果缓存没有的话，就把传入的事件存到缓存里，都可以理解为变成静态节点了，而在 Vue2 中是没有缓存的

# 静态标记

看一下静态标记是啥

```js
export const enum PatchFlags {
  TEXT = 1 ,  // 动态文本节点
  CLASS = 1 << 1,  // 2   动态class
  STYLE = 1 << 2,  // 4   动态style
  PROPS = 1 << 3,  // 8   除去class/style以外的动态属性
  FULL_PROPS = 1 << 4,       // 16  有动态key属性的节点，当key改变时，需进行完整的diff比较
  HYDRATE_EVENTS = 1 << 5,   // 32  有监听事件的节点
  STABLE_FRAGMENT = 1 << 6,  // 64  一个不会改变子节点顺序的fragment (一个组件内多个根元素就会用fragment包裹)
  KEYED_FRAGMENT = 1 << 7,   // 128 带有key属性的fragment或部分子节点有key
  UNKEYEN_FRAGMENT = 1 << 8, // 256  子节点没有key的fragment
  NEED_PATCH = 1 << 9,       // 512  一个节点只会进行非props比较
  DYNAMIC_SLOTS = 1 << 10,   // 1024   动态slot
  HOISTED = -1,  // 静态节点 
  BAIL = -2      // 表示 Diff 过程中不需要优化
}
```

```js
<div id="app">
    <div>node</div>
    <p>{{ age }}</p>
</div>
```

在 Vue2 中编译的结果是：

```js
with(this){
    return _c(
      'div',
      {attrs:{"id":"app"}},
      [ 
        _c('div',[_v("node")]),
        _c('p',[_v(_s(age))])
      ]
    )
}
```

在 Vue3 中编译的结果是这样的：

```js
const _hoisted_1 = { id: "app" }
const _hoisted_2 = /*#__PURE__*/_createElementVNode("div", null, "node", -1 /* HOISTED */)

export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", _hoisted_1, [
    _hoisted_2,
    _createElementVNode("p", null, _toDisplayString(_ctx.age), 1 /* TEXT */)
  ]))
}

```

上面编译结果中的 `-1` 和 `1` 了就是静态标记，这是在 Vue2 中没有的，patch 过程中就会判断这个标记来 Diff 优化流程，跳过一些静态节点对比

# 静态提升

其实还是拿上面 Vue2 和 Vue3 静态标记的例子，在 Vue2 里每当触发更新的时候，不管元素是否参与更新，每次都会全部重新创建，就是下面这一堆

```js
with(this){
    return _c(
      'div',
      {attrs:{"id":"app"}},
      [ 
        _c('div',[_v("node")]),
        _c('p',[_v(_s(age))])
      ]
    )
}

```

而在 Vue3 中会把这个不参与更新的元素保存起来，只创建一次，之后在每次渲染的时候不停地复用，比如上面例子中的这个，静态的创建一次保存起来

```js
const _hoisted_1 = { id: "app" }
const _hoisted_2 = /*#__PURE__*/_createElementVNode("div", null, "node", -1 /* HOISTED */)

```

然后每次更新 age 的时候，就只创建这个动态的内容，复用上面保存的静态内容

```js
export function render(_ctx, _cache, $props, $setup, $data, $options) {
  return (_openBlock(), _createElementBlock("div", _hoisted_1, [
    _hoisted_2,
    _createElementVNode("p", null, _toDisplayString(_ctx.age), 1 /* TEXT */)
  ]))
}

```

# patchKeyedChildren

在 Vue2 里 `updateChildren` 会进行

- 头和头比
- 尾和尾比
- 头和尾比
- 尾和头比
- 都没有命中的对比

在 Vue3 里 `patchKeyedChildren` 为

- 头和头比
- 尾和尾比
- 基于最长递增子序列进行移动/添加/删除

看个例子，比如

- 老的 children：`[ a, b, c, d, e, f, g ]`
- 新的 children：`[ a, b, f, c, d, e, h, g ]`

1. 先进行头和头比，发现不同就结束循环，得到 `[ a, b ]`
2. 再进行尾和尾比，发现不同就结束循环，得到 `[ g ]`
3. 再保存没有比较过的节点 `[ f, c, d, e, h ]`，并通过 newIndexToOldIndexMap 拿到在数组里对应的下标，生成数组 `[ 5, 2, 3, 4, -1 ]`，`-1` 是老数组里没有的就说明是新增
4. 然后再拿取出数组里的最长递增子序列，也就是 `[ 2, 3, 4 ]` 对应的节点 `[ c, d, e ]`
5. 然后只需要把其他剩余的节点，基于 `[ c, d, e ]` 的位置进行移动/新增/删除就可以了

使用最长递增子序列可以最大程度的减少 DOM 的移动，达到最少的 DOM 操作。有兴趣的话去 lc300 体验一下~
