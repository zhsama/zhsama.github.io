---
title: 浅谈vue3中的ref和reactive
date: 2022-03-25 14:21:32
updated: 2022-04-30 17:31:21
tags: ['vue3']
categories:
- vue3
---

# ref和reactive

`ref`本质也是`reactive`，`ref(obj)`等价于`reactive({value: obj})`：

* vue3中实现响应式数据的方法是就是使用`ref`和`reactive`，所谓响应式就是界面和数据同步，能实现实时更新

* vue2中响应式是通过`defineProperty`实现的，vue3中是通过ES6的`Proxy`来实现的
<!--more-->
## reactive

reactive的参数必须是一个对象，包括json数据和数组都可以，否则不具有响应式。如果给reactive传递了其他对象（如时间对象），默认情况下修改对象界面不会自动更新，如果想更新，可以通过给对象重新赋值来解决。

## ref

既然有了`reactive`，为何还要`ref`呢？当我们只想让某个变量实现响应式的时候，采用reactive就会比较麻烦，因此vue3提供了ref方法进行简单值的监听，但并不是说ref只能传入简单值，他的底层是reactive。

# shallowRef和shallowReactive

## 递归监听和非递归监听

1. `ref`和`reactive`：都属于递归监听，也就是数据的每一层都是响应式的，如果数据量比较大，非常消耗性能。
2. `shallowReactive`和`shallowRef`：都属于非递归监听，两个方法来包装响应式数据，通过该方法包装的数据只有第一层具有响应性。

## ref 和 shallowRef

- `ref`定义的数据每一层都是响应式数据
- `shallowRef`定义的数据，只有第一层是响应式的，即只有在更改`.value`的时候才能实现响应式

```typescript
let age = ref({
      a: '1',
      f: {
        b: '2',
        s:{
          c: '3'
        }
      }
    })
//打印各层数据
console.log(age);
console.log(age.value);
console.log(age.value.f);
console.log(age.value.f.s);
```

```js
let age = shallowRef({
      a: '1',
      f: {
        b: '2',
        s:{
          c: '3'
        }
      }
    })
//打印各层数据
console.log(age);
console.log(age.value);
console.log(age.value.f);
console.log(age.value.f.s);
```

使用`shallowRef`后，可以通过`triggerRef()`方法主动更新界面，实现界面刷新

```ts
function doSome(){
  age.value.f.s.c = 'c';
  //主动更新界面
  triggerRef(age);
}
```

## reactive 和 shallowReactive

- `reactive `定义的数据每一层都是响应式数据
- `shallowReactive` 定义的数据，只有第一层是响应式的

**注意：**`shallowReactive`没有类似 `triggerRef()` 的方法

# toRaw

有些时候我们不希望数据进行响应式实时更新，可以通过`toRaw`获取`ref`或`reactive`引用的原始数据，通过修改原始数据，不会造成界面的更新，只有通过修改`ref`和`reactive`包装后的数据时才会发生界面响应式变化。

```typescript
let obj1 = {...};
//state和obj1是引用关系，state的本质是一个Proxy对象，其中引用了obj1
let state = reactive(obj1);
//通过toRaw方法获取到原始数据，其实是获取到obj1的内存地址，obj2和obj1是完全相等的
let obj2 = toRaw(state)
console.log(obj1 === obj2);//true

```

有些同学会问，那直接使用obj1来修改数据不就行了吗？可关键是我们在使用`reactive`定义数据时一般不会先定义一个`obj`，再将他传给`reactive`，都是直接在`reactive`中写数据的。

# markRaw

与toRaw不同，markRaw包装后的数据永远不会被追踪。

```typescript
let obj1 = {name: "lijing", age: 18}
let obj2 = markRaw(obj1);
//此时reactive包装的数据虽然是响应式对象，但是不会被跟踪，也不会产生效应式效果
let state1 = reactive(obj2)

console.log(obj1 === obj2); //true
```

# toRef 和 toRefs

`ref`和`toRef`都是用来构造响应式数据，两者有什么区别呢，看两个例子

## ref

复制、修改响应式数据不会影响以前的数据，数据发生改变界面就会自动更新。转换后的是一个`RefImpl`类型

![image](https://user-images.githubusercontent.com/33454514/166103528-1211a22b-9b35-4ef6-9e5c-9b0b463f2ae4.png)

可以看到，使用ref对一个对象的某个简单数据类型属性进行响应式改造后，通过修改响应式数据不会影响到原始数据，如上图中，通过state1修改值后，obj1中的a属性值没有发生变化。

**注意点：**修改的这个属性必须是简单数据类型，一个具体的值，不能是引用，如果该属性也是一个对象，则会影响。

## toRef

ref类似深拷贝，toref类似浅拷贝。如果使用toRef来转换，则修改响应式数据会影响到原始数据，数据发生改变，但是界面不会自动更新。转换后的是一个`ObjectRefImpl`类型![image](https://user-images.githubusercontent.com/33454514/166103644-81bfcefe-f9d4-44c6-8261-e1caa2038f23.png)

## toRefs

遍历对象中的所有属性，将其变为响应式数据

![image](https://user-images.githubusercontent.com/33454514/166103685-851b36f2-663b-4f94-a728-054ad8bb3f12.png)
