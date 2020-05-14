# 初次使用
## Vue组合API
> cdn地址 https://s1.zhuanstatic.com/common/js/vue-next-3.0.0-alpha.0.js

### 一个简单的例子

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script src="https://s1.zhuanstatic.com/common/js/vue-next-3.0.0-alpha.0.js"></script>
</head>
<body>
  <div id="app"></div>
</body>
<script>
  const {createApp, reactive, computed, effect, onCreated, onMounted, ref, onUnmounted} = Vue
  const rootComponent = {
    template: `
    <button @click="increment">
      {{state.name}}今年{{state.age}}岁了, double是 {{state.double}}
    </button>
    `,
    setup() {
      const state = reactive({
        name: 'zsf',
        age: 0,
        double: computed(() => state.age * 2)
      })
      effect(() => {
        console.log(`effect 触发-${state.name}今年${state.age}岁`)
      })
      function increment() {
        state.age++
      }
      return {
        increment, state
      }
    }
  }
  createApp().mount(RootComponent, '#app')
</script>
</html>
```

### `setup`

setup 函数是一个新的组件选项。作为在组件内使用 Composition API 的入口点。

+ 调用时机
  
  去掉了vue2的 `beforeCreate` 和 `created`，统一使用 `setup`，会在 `beforeCreate` 钩子之前被调用

+ 参数

  有两个参数，分别是 `props` 和 `context`，`context` 会暴露原来 `this` 的一些属性，如：`attrs`、`slots`、`emit`。

> 注意：需要在模板中使用的数据或者方法，需要通过 `return` 暴露出来。

  ### `reactive`

  接收一个普通对象然后返回该普通对象的响应式代理。等同于 2.x 的 `Vue.observable()`

  响应式转换是“深层的”：会影响对象内部所有嵌套的属性。基于 ES2015 的 Proxy 实现，返回的代理对象不等于原始对象。建议仅使用代理对象而避免依赖原始对象。

  ### `ref`

  vue3 新加的 API，接受一个参数值并返回一个响应式且可改变的 ref 对象。ref 对象拥有一个指向内部值的单一属性 `.value`。

  ```
  const count = ref(0)
  ```

  当使用组合式 API 时，reactive refs 和 template refs 的概念已经是统一的。为了获得对模板内元素或组件实例的引用，我们可以像往常一样在 `setup()` 中声明一个 ref 并返回它：

  ```
  const RootComponent = {
    template: `
    <div ref="root">123</div>
    `,
    setup() {
      const root = ref(null)
      onMounted(() => {
        // 在渲染完成后, 这个 div DOM 会被赋值给 root ref 对象
        console.log(root.value) // <div>123</div>
      })
    }
  }
  ```
  
### `ref vs reactive`

```
// 风格 1: 将变量分离
let x = 0
let y = 0

function updatePosition(e) {
  x = e.pageX
  y = e.pageY
}

// --- 与下面的相比较 ---

// 风格 2: 单个对象
const pos = {
  x: 0,
  y: 0,
}

function updatePosition(e) {
  pos.x = e.pageX
  pos.y = e.pageY
}
```

组件内使用 `reactive` 时，返回的对象不能被解构或展开，否则会失去响应式，若想展开，可以使用 `toRefs(pos)`

### `computed`

除了例子中的使用方式，还可以传入 `get` `set` 函数对象，创建一个可手动修改的计算状态。

```
const state = reactive({
  age: 0,
  double: computed({
    get: () => state.age * 2,
    set: (val) =>  state.age = val / 2
  })
})

state.double = 2 // state.age = 1
```

### `生命周期`

+ ~~beforeCreate~~ -> 使用 `setup()`
+ ~~created~~ -> 使用 `setup()`
+ `beforeMount` -> `onBeforeMount`
+ `mounted` -> `onMounted`
+ `beforeUpdate` -> `onBeforeUpdate`
+ `updated` -> `onUpdated`
+ `beforeDestroy` -> `onBeforeUnmount`
+ `destroyed` -> `onUnmounted`
+ `errorCaptured` -> `onErrorCaptured`
