---
title: "Basics of Vue"
author: "Haobo Gu"
tags: [frontend]
date: 2022-04-25T19:57:40+08:00
summary: Vue入门学习
---

# Basics of Vue
什么是Vue？
>Vue是一款用于构建用户界面的 JavaScript 框架。它基于标准 HTML、CSS 和 JavaScript 构建，并提供了一套声明式的、组件化的编程模型，帮助你高效地开发用户界面，无论任务是简单还是复杂。

## 简介
Vue是现在最流行的前端框架之一，它主要拓展了HTML的语法，使得我们可以声明基于JS状态的HTML。同时，Vue会追踪JS的状态变化，并且响应式地更新HTML。Vue既可以直接增强静态的HTML而无需构建流程，也可以作为Web Component嵌入网页。

对于具有构建流程的工程来说，Vue支持一种特别的文件，名叫**单文件组件**（也就是`.vue`文件，英文缩写为SFC）。在SFC中，你可以把HTML、CSS、JavaScript封装到一个文件中，下面就是一个例子：
```vue
<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>

<template>
  <button @click="count++">Count is: {{ count }}</button>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>
```
可以看到整个文件分成三块，从上到下分别是JS、HTML和CSS。

## Vue基础
下面开始介绍Vue的基础知识，我们从创建一个Vue应用开始。
### Vue应用
#### 应用实例和根组件
每一个Vue应用实例都需要使用`createApp`创建，`createApp`的入参是**根组件**，根组件下面会有各种各样的子组件来构成页面。
```javascript
import { createApp } from 'vue'

const app = createApp({
  /* 根组件选项 */
})

// 我们也可以从其他文件中导入根组件
import { createApp } from 'vue'
// 从一个单文件组件中导入根组件
import App from './App.vue'

const app = createApp(App)
```
#### 挂载应用
每一个应用实例需要被挂载，即调用了`.mount()`函数之后才能被渲染。`.mount()`接收一个“容器”参数，可以是一个实际的 **DOM 元素**或是一个 **CSS 选择器**字符串：
```vue
<div id="app"></div>
```
```javascript
app.mount('#app')
```
这样，每一个页面就可以有多个Vue的应用实例，把应用挂载在对应的元素上即可

### 模板语法
> Vue 使用一种基于 HTML 的模板语法，使我们能够声明式地将其组件实例的数据绑定到呈现的 DOM 上。所有的 Vue 模板都是语法上合法的 HTML，可以被符合规范的浏览器和 HTML 解析器解析。

#### 文本插值
最简单的数据绑定形式就是文本插值，在HTML中使用双大括号来指定：
```vue
<span>Message: {{ msg }}</span>
```
在上面的例子中，双大括号标签会被替换为msg对应的property的值，每次`msg`的property的值改变的时候HTML渲染的结果也会实时更新。

#### [绑定Attributes](https://staging-cn.vuejs.org/guide/essentials/template-syntax.html#attribute-bindings)
HTML中，每个标签有各种Attributes。如果想把各种Attributes绑定到可变的值里面，可以用`v-bind`：
```vue
<div v-bind:id="dynamicId"></div>
```
上面的代码就把`<div>`的attribute `id`绑定在了`dynamicId`这个值上面。`v-bind:`可以使用`:`代替。如果你想绑定特定的class，把id替换成对应的class即可。下面就是一个例子：
```vue
<script setup>
import { ref } from 'vue'

const titleClass = ref('title')
</script>

<template>
  <h1 :class="titleClass">Make me red</h1>
</template>

<style>
.title {
  color: red;
}
</style>
```
这样，就把`Make me red`这个文本的颜色设置成了红色（即`<style>`中css定义的颜色）

#### [事件监听](https://staging-cn.vuejs.org/guide/essentials/event-handling.html)
我们可以使用`v-on`来监听DOM事件：
```vue
<button v-on:click="increment">{{ count }}</button>
```
也可以使用`@`作为简写：
```vue
<button @click="increment">{{ count }}</button>
```
上面的代码就监听了`<button>`的click事件，当监听到事件之后会触发`increment`函数。

#### [表单输入的绑定](https://staging-cn.vuejs.org/guide/essentials/forms.html#checkbox-2)
Vue对于表单输入的case有一种特殊的语法，叫做`v-model`。首先，我们看一下使用上面介绍的`v-on`和`v-bind`如何做到表单输入：
```vue
<input :value="text" @input="onInput">
```
```javascript
function onInput(e) {
  // a v-on handler receives the native DOM event
  // as the argument.
  text.value = e.target.value
}
```
在HTML的中，绑定一个值名为`text`，然后把`onInput`函数绑定在`input`事件上，最后在`onInput`中修改`text`的值。

Vue直接提供了`v-model`用来做这种事情，其用法也很简单：
```vue
<input v-model="text">
```
这样就自动把输入的值（即HTML中的`input`）绑定在了`text`上并且实现自动更新。`v-model`不只能绑定文本输入，还能绑定很多类型的输入比如checkbox、单选按钮等等。

#### [条件渲染](https://staging-cn.vuejs.org/guide/essentials/conditional.html)
在Vue中，可以使用条件渲染来决定某一个元素是否被渲染。条件渲染的关键词有三个：`v-if`,`v-else`,`v-else-if`。其中，`v-else`和`v-else-if`必须跟在一个`v-if`或一个`v-else-if`元素后面。`v-if`后面可以跟一个JavaScript语句，其值为true或者false，下面是一个例子：
```vue
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

在某些情况下，我们想要渲染一系列元素，但是给每个元素添加相同的`v-if`又太麻烦了，这时候我们应该使用`<template>`元素来把所有元素包起来：
```vue
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```
`<template>`元素并不会被渲染，它只是一个不可见的包装器元素。

除了`v-if`之外，Vue还提供了`v-show`来决定一个元素是否被显示。`v-show`和`v-if`的区别在于，`v-show`仅切换了该元素上名为`display`的CSS属性，无论该元素显示与否，它都会被渲染（而`v-if`直接决定该元素会不会被渲染）。同时`v-show`也不支持else。

总的来说，`v-if`在首次渲染时的切换成本比`v-show`更高。因此当你需要非常频繁切换时`v-show`会更好（因为`v-if`每次都会切换渲染 or 不渲染），而运行时不常改变的时候`v-if`会更合适。

#### [列表渲染](https://staging-cn.vuejs.org/guide/essentials/list.html)
我们可以使用`v-for`来渲染一个元素的列表，其语法如下：
```vue
<ul>
  <li v-for="todo in todos" :key="todo.id">
    {{ todo.text }}
  </li>  
</ul>
```
其中，`todos`是一个列表，`v-for`会遍历这个列表，然后把当前的值命名为`todo`，`todo`这个变量只能在`v-for`元素内部使用。同时，可以看到我们使用了`:key="todo.id"`给每个`todo`变量分配了一个id，并且绑定到了`key`上面。这个`key`是Vue中一个特殊的属性，可以让列表自动渲染到对应位置，可以参考这里：https://staging-cn.vuejs.org/api/built-in-special-attributes.html#key。

#### [计算属性](https://staging-cn.vuejs.org/guide/essentials/computed.html#basic-example)
当我们需要展示的某些值/属性需要计算之后才能得到结果时，可以使用`computed()`来完成对应的计算。下面是一个更加复杂的todo例子：
```vue
<script setup>
import { ref, computed } from 'vue'

let id = 0

const newTodo = ref('')
const hideCompleted = ref(false)
const todos = ref([
  { id: id++, text: 'Learn HTML', done: true },
  { id: id++, text: 'Learn JavaScript', done: true },
  { id: id++, text: 'Learn Vue', done: false }
])

const filteredTodos = computed(() => {
  return hideCompleted.value
    ? todos.value.filter((t) => !t.done)
    : todos.value
})

function addTodo() {
  todos.value.push({ id: id++, text: newTodo.value, done: false })
  newTodo.value = ''
}

function removeTodo(todo) {
  todos.value = todos.value.filter((t) => t !== todo)
}
</script>

<template>
  <input v-model="newTodo" @keyup.enter="addTodo">
  <button @click="addTodo">Add Todo</button>
  <ul>
    <li v-for="todo in filteredTodos" :key="todo.id">
      <input type="checkbox" v-model="todo.done">
      <span :class="{ done: todo.done }">{{ todo.text }}</span>
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
  <button @click="hideCompleted = !hideCompleted">
    {{ hideCompleted ? 'Show all' : 'Hide completed' }}
  </button>
</template>

<style>
.done {
  text-decoration: line-through;
}
</style>
```
在这个例子中，我们给每个todo分配一个`done`字段，代表是否完成；还声明了一个`hideCompleted`变量，当这个变量为true的时候，隐藏已经完成的todo。这样的话，在每一次点击hideCompleted按钮的时候，`computed()`都会根据当前的所有todo（即代码中的`todos`）计算出来`filteredTodos`，然后显示。

`computed()`会自动地追踪输入的状态，并且在状态有改变的时候自动地更新输出（即`filteredTodos`）。

### [模板ref](https://staging-cn.vuejs.org/guide/essentials/template-refs.html)
上面我们介绍了在Vue中常用的一些语法，当然在必要的时候我们也可以手动地去操作DOM。在HTML中，我们可以使用`ref="p"`来声明一个DOM对应的ref。在`<script setup>`中，我们也需要首先来声明对应的ref变量：
```javascript
const p = ref(null)
```
注意，此时我们使用`null`来声明这个变量，这是因为在`<script setup>`中这个DOM元素还没有被挂载。只有当这个组件被挂载之后，我们才能够在代码里面获取到对应的对象。这里就涉及到了[**生命周期**](https://staging-cn.vuejs.org/guide/essentials/lifecycle.html)，在这里我们需要实现一个`onMounted`钩子，就可以在组件被加载之后自动执行，并且获取到对应的模板ref：
```vue
<script setup>
import { ref, onMounted } from 'vue'

const p = ref(null)

onMounted(() => {
  p.value.textContent = 'mounted!'
})
</script>

<template>
  <p ref="p">hello</p>
</template>
```
这样，在组件加载之后就可以在代码中获取到其引用，并且自动地把文本换成"mounted!"

### [监听器](https://staging-cn.vuejs.org/guide/essentials/watchers.html)
监听器顾名思义就是监听某个ref，在它有变化的时候完成某些事件。在Vue中，可以使用`watch()`来监听某个ref：
```js
import { ref, watch } from 'vue'

const count = ref(0)

watch(count, (newCount) => {
  // 每当count变化，在控制台中打日志
  console.log(`new count is: ${newCount}`)
})
```
### 组件Component
在真实的代码中，每个应用都是多层组件嵌套而成的。我们可以从其他`.vue`文件中导入组件，并且直接使用：
```vue
<script setup>
import ChildComp from './ChildComp.vue'
</script>

<template>
  <ChildComp />
</template>
```

#### 组件的props
组件可以声明props属性，然后在父组件使用的时候，就可以传入对应的props作为输入。

首先，在定义组件的时候，我们需要使用`defineProps`声明其接受的props：
```vue
<!-- ChildComp.vue -->
<script setup>
const props = defineProps({
  msg: String
})
</script>
```
`defineProps`无需导入，可以直接使用。在定义完成之后，其父组件就可以向使用Attributes一样使用子组件定义的props，也可以使用`v-bind`来绑定一个变量值：
```vue
<ChildComp :msg="greeting" />
```

#### Emits
除了props之外，子组件可以向父组件发送事件，然后在父组件中使用`v-on`监听。和props类似，emits需要使用`defineEmits`定义，且无需导入：
```vue
<script setup>
const emit = defineEmits(['response'])

emit('response', 'hello from child')
</script>
```
这样，在父组件中就可以使用`v-on`来监听`response`事件了：
```vue
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const childMsg = ref('No child msg yet')

</script>
<template>
  <ChildComp @response="(msg) => childMsg = msg" />
  <p>{{ childMsg }}</p>
</template>
```
#### 传递slots
除了通过props的形式之外，父组件还可以通过slot的形式来向子组件传递数据：
```vue
<ChildComp>
  这里是slot
</ChildComp>
```
这就让父组件可以给子组件传递`<template>`数据了。在子组件中，也可以在对应的位置声明`<slot>`，作为默认值：
```vue
<template>
  <slot>Fallback content</slot>
</template>
```
然后当父组件什么都不传的时候，就会默认地显示`Fallback content`。而在父组件传递数据的时候，对应的位置会显示数据：
```vue
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const msg = ref('from parent')
</script>

<template>
  <ChildComp>{{msg}}</ChildComp>
</template>
```
此时，子组件这个地方会显示父组件里的`msg`内容，而不是`Fallback content`。

#### More
更多的Vue组件基础，可以参考：https://staging-cn.vuejs.org/guide/essentials/component-basics.html#components-basics