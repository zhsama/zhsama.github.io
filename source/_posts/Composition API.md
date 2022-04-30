---
title: Composition API
date: 2022-03-25 14:21:32
updated: 2022-04-30 17:31:21
tags: ['vue3']
categories:
- vue3
---

# Composition API

[官方网站](https://v3.cn.vuejs.org/guide/composition-api-introduction.html)

## setup

setup函数是一个新的组件选项。作为组件内使用Composition API的入口点

创建组件实例，然后初始化props，紧接着调用`setup`函数。它会在`beforeCreate`钩子之前调用。

setup返回一个对象。则对象的所有属性（**它是响应式的数据**）都可以直接在模板中使用。相当于vue2.0中data函数返回的对象。

```vue
<script>
export default {
  setup () {
    return {}
  }
}
</script>
```

## 响应式数据

简而言之，reactive负责复杂数据结构，ref可以把基本的数据结构包装成响应式：

- ref：可传入任意类型的值并返回一个响应式且可改变的ref对象。ref对象拥有一个指向内部值的单一属性`.value`，改变值的时候必须使用其value属性
- reactive：接受一个普通对象然后返回该普通对象的响应式代理。等同于2.x的`Vue.obserable()`

### ref

```vue
<template>
  <div>
    <h2>{{state.count}}</h2>
    <h3>{{num}}</h3>
    <button @click="add">计算</button>
  </div>
</template>

<script>
import { reactive, ref } from "vue";
export default {

  setup(){
    const state = reactive({
      count: 1
    });
    const num = ref(0);
    function add() {
      state.count++;
      num.value+=2
    }
    return { state, add, num };
  }
};
</script>
```

ref包装的num,模板里可以直接用，但js中修改的时候操作`.value`属性。

### reactive

```html
<template>
  <div>
    <h2>{{state.count}}</h2>
    <button @click="add">计算</button>
  </div>
</template>

<script>
import { reactive } from "vue";
export default {
  setup(){
    // 响应式变量声明 reactive负责复杂数据结构，
    const state = reactive({
      count: 1
    });
    function add() {
      state.count++;
    }
    return { state, add};
  }
};
</script>
```

### toRefs

将响应式对象转换为普通对象，其中结果对象的每个property都是指向原始对象相应的property。

从合成函数返回响应式对象时，toRefs非常有用，这样消费组件就可以在不丢失响应式的情况下对返回的对象进行分解/扩散：

```js
// useFeatureX.js
import {reactive} from 'vue';
export function userFeatureX(){
  const state = reactive({
    foo: 1,
    bar: 2
  })
  // 逻辑运行状态
  ...
  // 返回时转换为ref
  return state;
}

// App.vue
import {toRefs} from 'vue'
export default {
  setup(){
    const state = useFeatureX();
    return {
      ...toRefs(state)
    }
  }
}
```

### computed

传入一个 getter 函数，返回一个默认不可手动修改的 ref 对象。

```js
import { reactive, ref, computed } from "vue";
export default {

  setup() {
    // 1.响应式变量声明 reactive负责复杂数据结构，
    const state = reactive({
      count: 1
    });
    // 2.ref可以把基本的数据结构包装成响应式
    const num = ref(0);
    // 3.创建只读的计算属性
    const computedEven1 = computed(() => state.count % 2);
    // 4.创建可读可写的计算属性
    const computedEven2 = computed({
      get:()=>{
        return state.count % 2;
      },
      set: newVal=>{
        state.count = newVal;
      }
    })

    // 事件的声明
    function add() {
      state.count++;
      num.value += 2;
    }

    function handleClick() {
      computedEven2.value = 10;
    }

    return { state, add, num, computedEven1,computedEven2,handleClick };
  }
};

```

### watchEffect

立即执行传入的一个函数，并响应式追踪其依赖，并在其依赖变更时重新运行该函数。

```js
const num = ref(0)

watchEffect(() => console.log(count.value))
// -> 打印出 0

setTimeout(() => {
  count.value++
  // -> 打印出 1
}, 100)

```

1. 停止监听

   * 隐式停止

   当 `watchEffect` 在组件的 `setup()` 函数或生命周期钩子被调用时， 侦听器会被链接到该组件的生命周期，并在组件卸载时自动停止

   * 显示停止

   在一些情况下，也可以显示调用返回值来停止侦听

   ```js
   const stop = watchEffect(()=>{
     /*...*/
   })
   //停止侦听
   stop()
   
   ```

2. 清除副作用

   有时候副作用函数会执行一些异步的副作用，这些响应需要在其失效时来清除(即完成之前状态已改变了)。可以在侦听副作用传入的函数中接受一个`onInvalidate`函数作为参数，用来注册清理失效时的回调。当以下情况发生时，这个**失效回调**会被触发：

   - 副作用即将重新执行时
   - 侦听器被停止(如果在`setup()`或生命周期钩子函数中使用了`watchEffect`,则在卸载组件时)

   官网的例子：

   ```js
   watchEffect((onInvalidate) => {
     const token = performAsyncOperation(id.value)
     onInvalidate(() => {
       // id 改变时 或 停止侦听时
       // 取消之前的异步操作
       token.cancel()
     })
   })
   
   ```

案例：实现对用户输入“防抖”效果

```html
<template>
  <div>
    <input type="text"
           v-model="keyword">
  </div>
</template>

<script>
  import { ref, watchEffect } from 'vue'

  export default {
    setup() {
      const keyword = ref('')
      const asyncPrint = val => {
        return setTimeout(() => {
          console.log('user input: ', val)
        }, 1000)
      }
      watchEffect(
        onInvalidate => {
          //用户输入的时间间隔小于1秒，都会立刻清除掉定时，不输入结果。正因为这个，实现了用户防抖的功能，只在用户输入时间间隔大于1秒，才做打印
          const timer = asyncPrint(keyword.value)
          onInvalidate(() => clearTimeout(timer))
          console.log('keyword change: ', keyword.value)
        },
        // flush: 'pre'  watch() 和 watchEffect() 在 DOM 挂载或更新之前运行副作用，所以当侦听器运行时，模板引用还未被更新。
        //flush: 'post' 选项来定义，这将在 DOM 更新后运行副作用，确保模板引用与 DOM 保持同步，并引用正确的元素。
        {
          flush: 'post' // 默认'pre'，同步'sync'，'pre'组件更新之前
        }
      )

      return {
        keyword
      }
    }
  }
  // 实现对用户输入“防抖”效果
</script>

```

### watch

`watch` API 完全等效于 2.x `this.$watch` （以及 `watch` 中相应的选项）。`watch` 需要侦听特定的数据源，并在回调函数中执行副作用。默认情况是懒执行的，也就是说仅在侦听的源变更时才执行回调。

watch()接收的第一个参数被称作"数据源",它可以是：

- 一个返回任意值的getter函数
- 一个包装对象(可以是ref也可以是reactive包装的对象)
- 一个包含上述两种数据源的数组

第二个参数是回调函数。回调函数只有当数据源发生变动时才会被触发：

1. 侦听单个数据源

   ```js
   const state = reactive({count: 1});
   
   //侦听一个reactive定义的数据,修改count值时会触发 watch的回调
   watch(()=>state.count,(newCount,oldCount)=>{
     console.log('newCount:',newCount);  
     console.log('oldCount:',oldCount);
   })
   //侦听一个ref
   const num = ref(0);
   watch(num,(newNum,oldNum)=>{
     console.log('newNum:',newNum);  
     console.log('oldNum:',oldNum);
   })
   
   ```

2. 侦听多个数据源(数组)

   ```js
   const state = reactive({count: 1});
   const num = ref(0);
   // 监听一个数组
   watch([()=>state.count,num],([newCount,newNum],[oldCount,oldNum])=>{
     console.log('new:',newCount,newNum);
     console.log('old:',oldCount,oldNum);
   })
   
   ```

3. 侦听复杂的嵌套对象

   我们实际开发中，复杂数据随处可见， 比如：

   ```js
   const state = reactive({
     person: {
       name: '张三',
       fav: ['帅哥','美女','音乐']
     },
   });
   watch(
     () => state.person,
     (newType, oldType) => {
       console.log("新值:", newType, "老值:", oldType);
     },
     { deep: true }, // 立即监听
   );
   
   ```

如果不使用第三个参数`deep:true`， 是无法监听到数据变化的。 前面我们提到，默认情况下，watch 是**惰性的**，那什么情况下不是惰性的， 可以立即执行回调函数呢？其实使用也很简单， 给第三个参数中设置`immediate: true`即可

同时，watch和watchEffect在停止侦听，清除副作用（相应地onInvalidate会作为回调的第三个参数传入）等方面行为一致。

```html
<template>
  <div>
    <input type="text"
      v-model="keyword">
  </div>
</template>

<script>
import { ref, watch } from 'vue'

export default {
  setup() {
    const keyword = ref('')
    const asyncPrint = val => {
      return setTimeout(() => {
        console.log('user input: ', val)
      })
    }

    watch(
      keyword,
      (newVal, oldVal, onCleanUp) => {
        const timer = asyncPrint(keyword)
        onCleanUp(() => clearTimeout(timer))
      },
      {
        lazy: true // 默认false，即初始监听回调函数执行了
      }
    )
    return {
      keyword
    }
  }
}
</script>

```

## 生命周期钩子

与2.x版本生命周期相对应的组合式API

| 选项式 API        | Hook inside `setup` |
| ----------------- | ------------------- |
| `beforeCreate`    | Not needed*         |
| `created`         | Not needed*         |
| `beforeMount`     | `onBeforeMount`     |
| `mounted`         | `onMounted`         |
| `beforeUpdate`    | `onBeforeUpdate`    |
| `updated`         | `onUpdated`         |
| `beforeUnmount`   | `onBeforeUnmount`   |
| `unmounted`       | `onUnmounted`       |
| `errorCaptured`   | `onErrorCaptured`   |
| `renderTracked`   | `onRenderTracked`   |
| `renderTriggered` | `onRenderTriggered` |
| `activated`       | `onActivated`       |
| `deactivated`     | `onDeactivated`     |

新建测试组件`/components/Test.vue`

```vue
<template>
  <div id="test">
    <h3>{{a}}</h3>
    <button @click="handleClick">更改</button>
  </div>
</template>

<script>
import {
  ref,
  onMounted,
  onBeforeMount,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
} from "vue";
export default {
  // 初始化数据阶段的生命周期，介于beforeCreate和created之间
  setup() {
    const a = ref(0);
    console.log("👌");
    function handleClick() {
      a.value += 1;
    }
    onBeforeMount(() => {
      console.log("组件挂载之前");
    });
    onMounted(() => {
      console.log("DOM挂载完成");
    });
    onBeforeUpdate(() => {
      console.log("DOM更新之前", document.getElementById("test").innerHTML);
    });
    onUpdated(() => {
      console.log("DOM更新完成", document.getElementById("test").innerHTML);
    });
    onBeforeUnmount(() => {
      console.log("实例卸载之前");
    });
    onUnmounted(() => {
      console.log("实例卸载之后");
    });
    return { a, handleClick };
  }
};
</script>
```

按照官方上说的那样，你不需要立马弄明白所有的东西，不过随着你的不断学习和使用，它的参考价值会越来越高。

![image](https://user-images.githubusercontent.com/33454514/166104036-60b9ff1b-e0bc-4537-bb5e-bc92011e84a7.png)

## 依赖注入

`provide`和`inject`提供依赖注入，功能类似2.x的`provide/inject`。两者都只能在当前组件的`setup()`中调用

`App.vue`provide数据源

```html
<template>
  <div>
    <Article></Article>
  </div>
</template>

<script>
import {
  ref,
  provide
} from "vue";
import Article from "./components/Article";
export default {
  setup() {
    const articleList = ref([
      { id: 1, title: "Vue3.0学习", author: "小马哥" },
      { id: 2, title: "componsition api", author: "尤大大" },
      { id: 3, title: "Vue-router最新", author: "vue官方" }
    ]);
    /* 
      provide 函数允许你通过两个参数定义 property：
      property 的 name (<String> 类型)
      property 的 value
    */
    provide("list",articleList);
    return {
      articleList
    };
  },
  components: {
    Article
  }
};
</script>

```

`Article.vue`注入数据

```html
<template>
  <div>
    {{articleList[0].title}}
  </div>
</template>

<script>
import { inject } from "vue";
export default {
  setup() {
    const articleList = inject('list',[]);
    return {articleList};
  },
};
</script>

```

## 模板引用refs

当使用组合式API时，`reactive refs`和`template refs`的概念已经是统一了。为了获得对模板内元素或者组件实例的引用，可以直接在`setup()`中声明一个ref并返回它

```vue
<template>
  <div>
    <div ref='wrap'>hello vue3.0</div>
    <Article ref='articleComp'></Article>
  </div>
</template>

<script>
import {
  ref,
  onMounted,
  provide
} from "vue";
import Article from "./components/Article";
export default {
  setup() {
    const isShow = ref(true);
    const wrap = ref(null);
    const articleComp = ref(null);

    const articleList = ref([
      { id: 1, title: "Vue3.0学习", author: "小马哥" },
      { id: 2, title: "componsition api", author: "尤大大" },
      { id: 3, title: "Vue-router最新", author: "vue官方" }
    ]);
    /* 
      provide 函数允许你通过两个参数定义 property：
      property 的 name (<String> 类型)
      property 的 value
    */
    provide("list", articleList);

    onMounted(() => {
      console.log(wrap.value); //获取div元素
      console.log(articleComp.value); //获取的article组件实例对象
      
    });
    return {
      articleList,
      wrap,
      articleComp
    };
  },
  components: {

    Article
  }
};
</script>

<style scoped>
</style>
```

## 组件通信

- props
- $emit
- expose /ref
- attrs
- v-model
- provide/inject
- vuex
- mitt

### props

```vue
// Parent.vue 
<child :msg1="msg1"
       :msg2="msg2">
</child>

<script>
import child from "./child.vue"
import { ref, reactive } from "vue"
export default {
    setup(){
        const msg1 = ref("这是传级子组件的信息1");
        const msg2 = reactive(["这是传级子组件的信息2"]);
        return {
            msg1,
            msg2
        }
    }
}
</script>

// Child.vue 
<script>
export default {
  props: ["msg1", "msg2"],// 如果这行不写，下面就接收不到
  setup(props) {
    console.log(props) // { msg1:"这是传给子组件的信息1", msg2:"这是传给子组件的信息2" }
  },
}
</script>

```

### $emit

```vue
// Child.vue
<template>
  <button @click="$emit('myClick',123)">按钮</buttom>
</template>
<script> 
 export default {
   emits:['myClick']
}

</script>

// Parent.vue 
<template>
    <child @myClick="onMyClick"></child>
</template>
<script setup>
import child from "./child.vue"

const onMyClick = (msg) => {
  console.log(msg)
}
</script>
```
