 
## 介绍

vue.js优势是（视图-模型）双向绑定，简化了dom的操作（不用重写大量的html标签），提高dom的复用率（以最少代码实现更多的功能），倾向于数据读写，虽然看上去使用比较繁琐，但是利于后期的维护。 



## 挂载点的选择el

只在由 new 创建的实例中遵守。
提供一个在页面上已存在的 DOM 元素作为 Vue 实例的挂载目标。可以是 CSS 选择器，也可以是一个 HTMLElement 实例。
在实例挂载之后， 元素可以用 vm.$el 访问。

> 提供的元素只能作为挂载点。不同于 Vue 1.x，所有的挂载元素会被 Vue 生成的 DOM 替换。因此不推荐挂载root实例到 `<html>` 或者 `<body>` 上。


## 数据绑定

数据绑定最常见的形式就是使用 “Mustache” 语法（双大括号）的文本插值

通过使用 `v-once` 指令，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上所有的数据绑定

双大括号会将数据解释为纯文本，而非 HTML 。为了输出真正的 HTML ，你需要使用 `v-html` 指令
被插入的内容都会被当做 HTML —— 数据绑定会被忽略。注意，你不能使用 v-html 来复合局部模板，因为 Vue 不是基于字符串的模板引擎。组件更适合担任 UI 重用与复合的基本单元。

```html
<span>Message: {{ msg }}</span>
<span v-once>This will never change: {{ msg }}</span>
<div v-html="rawHtml"></div>
```
## 指令
 
 指令是特殊的带有前缀 v- 特性。指令的值限定为 绑定表达式 ，它的职责是当其表达式的值改变时把某些行为应用到DOM上。

### v-show 、v-if

一般来说， v-if 有更高的切换消耗而 v-show 有更高的初始渲染消耗。因此，如果需要频繁切换 v-show 较好，如果在运行时条件不大可能改变 v-if 较好

### v-for

```html

<ul id="repeat-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>

<!--  你也可以提供第二个的参数为键名： -->

<div v-for="(value, key) in object">
  {{ key }} : {{ value }}
</div> 

<!-- 第三个参数为索引：-->

<div v-for="(value, key, index) in object">
  {{ index }}. {{ key }} : {{ value }}
</div>
```

### v-on／@
利用 v-on 指令用于监听DOM事件，例如： `<a v-on:click="doSomething">`

### v-bind／:

利用 v-bind 指令用于响应的更新HTML属性，例如： `<a v-bind:href="url"></a>`
 

### 自定义指令

除了默认设置的核心指令( v-model 和 v-show ),Vue 也允许注册自定义指令。注意，在 Vue2.0 里面，代码复用的主要形式和抽象是组件——然而，有的情况下,你仍然需要对纯 DOM 元素进行底层操作,这时候就会用到自定义指令。下面这个例子将聚焦一个 input 元素，像这样：

```javascript
// 注册一个全局自定义指令 v-focus
Vue.directive('focus', {
  // 当绑定元素插入到 DOM 中。
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})
```
也可以注册局部指令，组件中接受一个 directives 的选项：

```javascript
directives: {
  focus: {
    // 指令的定义---
  }
}
```
然后你可以在模板中任何元素上使用新的 v-focus 属性：
`<input v-focus>`

## keep-alive
`<keep-alive>` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。和 `<transition>` 相似，`<keep-alive>` 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。

当组件在 `<keep-alive>` 内被切换，它的 `activated` 和 `deactivated` 这两个生命周期钩子函数将会被对应执行。

主要用于保留组件状态或避免重新渲染。


## 深入响应式原理

Vue 最显著的特性之一便是不太引人注意的响应式系统(reactivity system)。模型层(model)只是普通 JavaScript 对象，修改它则更新视图(view)。这会让状态管理变得非常简单且直观，不过理解它的工作原理以避免一些常见的问题也是很重要的。

把一个普通 Javascript 对象传给 Vue 实例的 data 选项，Vue 将遍历此对象所有的属性，并使用`Object.defineProperty` 把这些属性全部转为 `getter/setter`。

用户看不到 getter/setter，但是在内部它们让 Vue 追踪依赖，在属性被访问和修改时通知变化。这里需要注意的问题是浏览器控制台在打印数据对象时 getter/setter 的格式化并不同，所以你可能需要安装 vue-devtools 来获取更加友好的检查接口。

** devtools **

开发版本默认为 `true`，生产版本默认为 `false`。生产版本设为 `true`可以启用检查。
`Vue.config.devtools = (process.env.NODE_ENV !== 'production')`



### set使用

受现代 Javascript 的限制（以及废弃 Object.observe），Vue 不能检测到对象属性的添加或删除。由于 Vue 会在初始化实例时对属性执行 `getter/setter` 转化过程，所以属性必须在 `data` 对象上存在才能让 Vue 转换它，这样才能让它是响应的。

```javascript
var vm = new Vue({
  data:{
  a:1
  }
})
// `vm.a` 是响应的
vm.b = 2
// `vm.b` 是非响应的
```
Vue 不允许在已经创建的实例上动态添加新的根级响应式属性(root-level reactive property)。然而它可以使用 Vue.set(object, key, value) 方法将响应属性添加到嵌套的对象上：
 
`this.$set(this.someObject,'b',2) ` 

用法：

设置对象的属性。如果对象是响应式的，确保属性被创建后也是响应式的，同时触发视图更新。这个方法主要用于避开 **Vue 不能检测属性被添加的限制**。

注意对象不能是 Vue 实例，或者 Vue 实例的根数据对象

参考： [深入响应式原理](https://cn.vuejs.org/v2/guide/reactivity.html)

 有时你想向已有对象上添加一些属性，例如使用 Object.assign() 或 _.extend() 方法来添加属性。但是，添加到对象上的新属性不会触发更新。在这种情况下可以创建一个新的对象，让它包含原对象的属性和新的属性：

```javascript
// 代替 `Object.assign(this.someObject, { a: 1, b: 2 })`
this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
```



 Vue 不能检测以下变动的数组：
1.当你利用索引直接设置一个项时，例如： vm.items[indexOfItem] = newValue
2.当你修改数组的长度时，例如： vm.items.length = newLength

为了避免第一种情况，以下两种方式将达到像 vm.items[indexOfItem] = newValue 的效果， 同时也将触发状态更新

``` javascript
// Vue.set
Vue.set(example1.items, indexOfItem, newValue)
// Array.prototype.splice`
example1.items.splice(indexOfItem, 1, newValue)

```

避免第二种情况，使用 splice：

``` javascript
example1.items.splice(newLength)
```


###  vm.$nextTick()


可能你还没有注意到，Vue 异步执行 DOM 更新。只要观察到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据改变。如果同一个 watcher 被多次触发，只会一次推入到队列中。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作上非常重要。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际（已去重的）工作。Vue 在内部尝试对异步队列使用原生的 Promise.then 和 MutationObserver，如果执行环境不支持，会采用 setTimeout(fn, 0) 代替。
例如，当你设置 vm.someData = 'new value' ，该组件不会立即重新渲染。当刷新队列时，组件会在事件循环队列清空时的下一个“tick”更新。多数情况我们不需要关心这个过程，但是如果你想在 DOM 状态更新后做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员沿着“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们确实要这么做。为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用 Vue.nextTick(callback) 。这样回调函数在 DOM 更新完成后就会调用。


``` javascript
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: 'not updated'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = 'updated'
      console.log(this.$el.textContent) // => '没有更新'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => '更新完成'
      })
    }
  }
})
```

### watch
为了发现对象内部值的变化，可以在选项参数中指定 deep: true 。注意监听数组的变动不需要这么做。

``` javascript
var vm = new Vue({
  data: {
    a: 1,
    b: 2,
    c: 3
  },
  watch: {
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
    // 方法名
    b: 'someMethod',
    // 深度 watcher
    c: {
      handler: function (val, oldVal) { /* ... */ },
      deep: true
    }
  }
})
vm.a = 2 // -> new: 2, old: 1

```


## 父子组件通信

### 使用Prop 传递数据

组件实例的作用域是孤立的。这意味着不能并且不应该在子组件的模板内直接引用父组件的数据。可以使用 props 把数据传给子组件。

prop 是父组件用来传递数据的一个自定义属性。子组件需要显式地用 `props` 选项声明 “prop”：

```javascript
Vue.component('child', {
  // 声明 props
  props: ['message'],
  // 就像 data 一样，prop 可以用在模板内
  // 同样也可以在 vm 实例中像 “this.message” 这样使用
  template: '<span>{{ message }}</span>'
})
```
然后向它传入一个普通字符串：

``` html
<child message="hello!"></child>

```
如果是动态属性 

``` html
<child :message="hello!"></child>

```

### 自定义事件

父组件是使用 props 传递数据给子组件，但如果子组件要把数据传递回去，应该怎样做？那就是自定义事件！

使用 `$on(eventName)` 监听事件
使用 `$emit(eventName)` 触发事件

下面是一个例子：

```html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```

```javascript
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})


new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```
 

### 单向数据流

prop 是单向绑定的：当父组件的属性变化时，将传导给子组件，但是不会反过来。这是为了防止子组件无意修改了父组件的状态——这会让应用的数据流难以理解。

另外，每次父组件更新时，子组件的所有 prop 都会更新为最新值。这意味着你**不应该**在子组件内部改变 prop 。如果你这么做了，Vue 会在控制台给出警告。

通常有两种改变 prop 的情况：
1.prop 作为初始值传入，子组件之后只是将它的初始值作为本地数据的初始值使用；
2.prop 作为需要被转变的原始值传入。
更确切的说这两种情况是：

```javascript
// 1.定义一个局部 data 属性，并将 prop 的初始值作为局部数据的初始值。
props: ['initialCounter'],
data: function () {
  return { counter: this.initialCounter }
}
// 2.定义一个 computed 属性，此属性从 prop 的值计算得出。
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```
 

### 异步组件

在大型应用中，我们可能需要将应用拆分为多个小模块，按需从服务器下载。为了让事情更简单， Vue.js 允许将组件定义为一个工厂函数，动态地解析组件的定义。Vue.js 只在组件需要渲染时触发工厂函数，并且把结果缓存起来，用于后面的再次渲染。


```javascript
Vue.component('async-webpack-example', function (resolve) {
  // 这个特殊的 require 语法告诉 webpack
  // 自动将编译后的代码分割成不同的块，
  // 这些块将通过 Ajax 请求自动下载。
  require(['./my-async-component'], resolve)
})

```
比如 [路由](./vue使用总结.html#vue-router)

当打包构建应用时，Javascript包会变得非常大，影响页面加载。如果我们能把不同路由对应的组件分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加高效了。


这种异步加载组件的写法 会生成类似 `11.11.js` 这样的的文件。在需要渲染的时候再加载


##扩展 

### 路由 vue-router

#### 使用步骤
```javascript
//  如果使用模块化机制编程，導入Vue和VueRouter，要调用 Vue.use(VueRouter)
import VueRouter from 'vue-router' 
Vue.use(VueRouter)


//  定义（路由）组件。
// 可以从其他文件 import 进来
const loanApply = resolve => require(['./loanApply/index'], resolve)

const loanConfirm = resolve => require(['./loanConfirm/index'], resolve)
 
//  创建 router 实例  
const router = new VueRouter({
  mode: 'hash', 
    base: __dirname,
    routes: [{
        path: '/', 
        component: loanApply,
        name: 'apply',
        meta: {
            title: '取点花'
        }
    }
}) 
// 创建和挂载根实例。
new Vue({
    el: '#app',
    router, 
    render: h => h(App)
})
```

#### 导航钩子

``` javascript
router.beforeEach((to, from, next) => {
  // to 和 from 都是 路由信息对象
})
```

[->更多](https://router.vuejs.org/zh-cn/essentials/getting-started.html)

### vuex

每一个 Vuex 应用的核心就是 store（仓库）。"store" 基本上就是一个容器，它包含着你的应用中大部分的**状态(state)**。Vuex 和单纯的全局对象有以下两点不同：

1.Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。

2.你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地**提交(commit)** mutations。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
// import { state, mutations } from './mutations'
Vue.use(Vuex)

const store = new Vuex.Store({
   strict: process.env.NODE_ENV !== 'production',//在严格模式下，无论何时发生了状态变更且不是由 mutation 函数引起的，将会抛出错误。这能保证所有的状态变更都能被调试工具跟踪到。
    state: { 
        data: '' 
    },
    mutations: { 
        saveData(state, obj) { 
            state.data = obj
 
        }
    }
})
```
现在，你可以通过 `store.state` 来获取状态对象，以及通过 `store.commit` 方法触发状态变更

<!-- 
import {
    mapState
} from 'vuex'
computed:{
  ...mapState({
            contacts: state => state.contacts
    })
}
 -->

  
** 赋值 **
`this.$store.commit('saveData', obj)`

** 取值 **
`this.$store.state.data`

再次强调，我们通过提交 mutation 的方式，而非直接改变 `store.state.count`，是因为我们想要更明确地追踪到状态的变化。这个简单的约定能够让你的意图更加明显，这样你在阅读代码的时候能更容易地解读应用内部的状态改变。此外，这样也让我们有机会去实现一些能记录每次状态改变，保存状态快照的调试工具。有了它，我们甚至可以实现如时间穿梭般的调试体验。

由于 store 中的状态是响应式的，在组件中调用 store 中的状态简单到仅需要在计算属性中返回即可。触发变化也仅仅是在组件的 methods 中提交 mutations。

[-> 更多](https://vuex.vuejs.org/zh-cn/state.html)



## 注意 



1.顶层的元素不能是组件  
2.组件模板应只包含一个根元素 




