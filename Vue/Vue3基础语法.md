# 组合式 API (Composition API)
组合式 API (Composition API) 是一系列 API 的集合，使我们可以使用函数而不是声明选项的方式书写 Vue 组件。

可以通过 app.config 配置应用级选项

Vue2.7及之后版本可以通过`@vue/composition-api`使用组合式API。

# 响应式状态
## 1. ref
ref() 接收参数，并将其包裹在一个带有 .value 属性的 ref 对象中返回，新建响应式对象。
- 模板中使用,不用加`.value`。( <a href="#unref">ref解包</a> )
- Ref可以劫持任何类型的值，包括深层嵌套的对象、数组等（Map）。
- 可以用`shallowRef`取消深层响应


```Vue3
使用：
const count = ref(0)

console.log(count) // { value: 0 }
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

<details>
<summary>setup()函数中使用</summary>

`setup()`钩子是在组件中使用组合式 API 的入口，通常只在以下情况下使用：
- 需要在非单文件组件中使用组合式 API 时（一个页面中有多个组件时）。
- 需要在基于选项式 API 的组件中集成基于组合式 API 的代码时。

```
import { ref } from 'vue'

export default {
  // `setup` 是一个特殊的钩子，专门用于组合式 API。
  setup() {
    const count = ref(0)

    // 将 ref 暴露给模板
    return {
      count
    }
  }
}
```

</details>


<h3 id="unref"> ref 解包: </h3>

所谓解包就是获取到 ref 对象上 value 属性的值。常用的两种方法就是 .value 和 unref()。
- 在模板渲染上下文中，只有顶级的`ref`会被解包(可以通过解构获取为顶级) // 3.2+
- 在模板中，`ref`为插值最终计算值，将被解包(eg: `{{ object.id }}`)
- 只有当嵌套在一个深层响应式对象内时，才会发生 ref 解包
    (eg: `let num = ref(0);reactive(num)`会自动解包)
- 在`computed`(计算属性)中`ref`也会自动解包
- 作为响应式数组或原始集合类型(`Map`)中的元素被访问时,不会解包


**unref():** 如果参数是 ref，则返回内部值，否则返回参数本身。这是`val = isRef(val) ? val.value : val`计算的一个语法糖。

ref 的解包: https://segmentfault.com/a/1190000043023603

响应式 API：工具函数：https://cn.vuejs.org/api/reactivity-utilities.html

### toRef
可以将值、refs 或 getters 规范化为 refs (3.3+)。

也可以基于响应式对象上的一个属性，创建一个对应的`ref`。这样创建的`ref`与其源属性保持同步：改变源属性的值将更新`ref`的值，反之亦然。
主要用法：创建一个新ref对象，基于旧ref对象的子属。
```
// 创建一个只读的 ref，当访问 .value 时会调用此 getter 函数 3.3+
toRef(() => props.foo)

const state = reactive({
  foo: 1,
  bar: 2
})

// 双向 ref，会与源属性同步
const fooRef = toRef(state, 'foo')

// 更改该 ref 会更新源属性
fooRef.value++
console.log(state.foo) // 2

```

### toRefs
将一个响应式对象转换为一个普通对象，这个普通对象的每个属性都是指向源对象相应属性的 ref。每个单独的 ref 都是使用 toRef() 创建的。

```
const state = reactive({
  foo: 1,
  bar: 2
})

const stateAsRefs = toRefs(state)

// 这个 ref 和源属性已经“链接上了”
state.foo++
console.log(stateAsRefs.foo.value) // 2

stateAsRefs.foo.value++
console.log(state.foo) // 3

```

## 2. reactive()
reactive() 将使对象本身具有响应性，使原始对象具有响应性。
- 也可以进行深层包装，`ref`在包装对象时也会使用`reactive`。
- 可以使用`shallowReactive()`退出深层响应式。
- 返回值为原始对象的`Proxy`,和原始对象不对等。
- 嵌套使用`reactive`会返回自身 </br>

缺点
- 只能用于对象类型
- 不能替换整个对象，会造成响应式丢失
- 对解构操作不友好，解构后会断开响应式链接

    <details>
    <summary>解构不友好例子</summary>

    ```Vue3
    // 当启用中间值后解构，将失去响应式
    const state = reactive({ count: 0 })

    // 当解构时，count 已经与 state.count 断开连接
    let { count } = state
    // 不会影响原始的 state
    count++

    // 该函数接收到的是一个普通的数字
    // 并且无法追踪 state.count 的变化
    // 我们必须传入整个对象以保持响应性
    callSomeFunction(state.count)
    ```
    </details>

# 监听
## watchEffect()
立即运行一个函数，同时响应式地追踪其依赖，并在依赖更改时重新执行，默认在组件渲染前执行。
- watchEffect 默认监听，也就是默认第一次就会执行；
- 不需要设置监听的数据，在 effect 函数中，用到了哪个数据，会自动进行依赖，因此不用担心类似 watch 中出现深层属性监听不到的问题；
- 只能获取到新值，由于没有提前指定监听的是哪个数据，所以不会提供旧值。
- 不能在`watchEffect`中使用异步

语法：watchEffect(effect, options)

- effect: 函数。内部依赖的响应式数据发生变化时，会触发 effect 重新执行
    - onCleanup：形参，函数类型，接受一个回调函数。每次更新时，会调用上一次注册的 onCleanup 函数。作用同 watch 中的 onCleanup 参数。
- options：
    - flush：pre | post | sync
        - pre：在组件更新前执行副作用；
        - post：在组件更新后运行副作用，可以使用 watchPostEffect 替代；
        - sync：每个更改都强制触发 watch，可以使用 watchSyncEffect 替代。
    - onTrack：函数，具备 event 参数，调试用。将在响应式 property 或 ref 作为依赖项被追踪时被调用
    - onTrigger：函数，具备 event 参数，调试用。将在依赖项变更导致副作用被触发时被调用。

```

function watchEffect(
  effect: (onCleanup: OnCleanup) => void,
  options?: WatchEffectOptions
): StopHandle

// 清除副作用：OnCleanup在下次执行前调用，可以清除上个周期中还没执行完的Api
// cleanupFn 内请求需要为可停止的，可借助 axios
type OnCleanup = (cleanupFn: () => void) => void

interface WatchEffectOptions {
  // 默认pre组件渲染前执行，post是渲染后，sync是响应式依赖改变时立即触发
  flush?: 'pre' | 'post' | 'sync' // 默认：'pre'
  onTrack?: (event: DebuggerEvent) => void
  onTrigger?: (event: DebuggerEvent) => void
}

// 取消监听，调用watchEffect返回的函数
type StopHandle = () => void

```

## watch
侦听一个或多个响应式数据源，并在数据源变化时调用所给的回调函数。

语法：watch(source, callback, options)
- source: 需要监听的响应式数据或者函数
- callback：监听的数据发生变化时，会触发 callback
    - newValue：数据的新值
    - oldValue：数据的旧值
    - onCleanup：函数类型，接受一个回调函数。每次更新时，会调用上一次注册的 onCleanup 函数
- options：额外的配置项
    - immediate：Boolean类型，是否在第一次就触发 watch
    - deep：Boolean 类型，是否开启深度监听
    - flush：pre | post | sync
        - pre：在组件更新前执行副作用
        - post：在组件更新后运行副作用
        - sync：每个更改都强制触发 watch
    - onTrack：函数，具备 event 参数，调试用。将在响应式 property 或 ref 作为依赖项被追踪时被调用
    - onTrigger：函数，具备 event 参数，调试用。将在依赖项变更导致副作用被触发时被调用。


参考：
Vue3：watch 的使用场景及常见问题：https://juejin.cn/post/7126364048886595598

# 常用指令
[vue3 指令] https://cn.vuejs.org/api/built-in-directives.html
## 1. v-bind(缩写：:)
### 1. 响应式绑定一个 attribute: 
<span id="moreProps"></span>


    ```
    // 方式1：绑定声明过的值
    <div v-bind:id="id"></div>
    <div :id="id"></div> // 简写
    <div :id></div> // 同名简写 3.4+
    
    // 方式2：布尔型 Attribute
    // 布尔型 attribute 依据 true / false 值来决定 attribute 是否应该存在于该元素上。disabled 就是最常见的例子之一。
    <button :disabled="isButtonDisabled> Button </button>

    // 方式3：动态绑定多个值
    const objectOfAttrs = {
        id: 'container',
        class: 'wrapper',
        style: 'background-color:green;'
    }
    <div v-bind="objectOfAttrs"></div>`

    相当于:
    <div :id="objectOfAttrs.id" :class="objectOfAttrs.class" :style="objectOfAttrs.style"></div>
    ```
#### attribute穿透(非响应式，不能监听)：
- 默认子组件的为**单个**根节点会继承父组件`attribute`,可以设置`defineOptions({inheritAttrs: false})`禁止。
- 可以通过`$attrs`获取父组件穿透的`attrubute`
- 这个 `$attrs` 对象包含了除组件所声明的 `props` 和 `emits` 之外的所有其他 `attribute`，例如 `class`，`style`，`v-on` 监听器等等。
- 多个根节点没有自动穿透，没有显示绑定会抛出警告。

```

// 运用 useAttrs 访问所有穿透
<script setup>
import { useAttrs } from 'vue'
const attrs = useAttrs()
</script>


```

### 2. 动态参数（推荐使用计算属性进行替换复杂表达式）
- 动态表达式的值只能是`null`或一个`字符串`
- 不能有空格和引号
    
    ```
    // attributeName为动态，若attributeName为href，则等价于 v-bind:href
    <a v-bind:[attributeName]="url">...</a>
    <a :[attributeName]="url">...</a>

    // 动态事件绑定：eventName 为动态设置
    <button v-on:[eventName]="doSomething"></button>
    <button @[eventName]="doSomething"></button>
    ```

## 2. v-on(缩写：@)
给元素绑定事件监听器(常用:@click,@input,@focus,@submit)
```
<button v-on:click="clear">Button</button>
<button @click="clear">Button</button>
```

### 修饰符
- `.stop` - 禁止冒泡，调用 event.stopPropagation()。
- `.prevent` - 禁止默认事件，调用 event.preventDefault()。
- `.capture` - 在捕获模式添加事件监听器,捕获是从最外层到内层。
- `.self` - 只有事件从元素本身发出才触发处理函数。
- `.{keyAlias}` - 只在某些按键下触发处理函数。
- `.once` - 最多触发一次处理函数。
- `.left` - 只在鼠标左键事件触发处理函数。
- `.right` - 只在鼠标右键事件触发处理函数。
- `.middle` - 只在鼠标中键事件触发处理函数。
- `.passive` - 通过 { passive: true } 附加一个 DOM 事件。

    <details>
    <summary>.{keyAlias} 使用</summary>

        // 全部键名：
        //    .enter .tab .delete .esc .space .up .down 
        //    .left .right .alt .ctrl .shift .meta 

        // 使用方式：
            // 单个：
            <button @keyup.86="clear"></button>
            // 模块需要获取焦距，再按下 alt + 任意键
            <button @keyup.alt="clear"></button> 
            
            // 多个：
            // 模块需要获取焦距，再按下 alt + v
            <button @keyup.alt.86="clear"></button> 

        // .exact:允许精确控制触发事件所需的系统修饰符的组合。
            <!-- 当按下 Ctrl 时，即使同时按下 Alt 或 Shift 也会触发 -->
            <button @click.ctrl="onClick">A</button>

            <!-- 仅当按下 Ctrl 且未按任何其他键时才会触发 -->
            <button @click.ctrl.exact="onCtrlClick">A</button>

            <!-- 仅当没有按下任何系统按键时触发 -->
            <button @click.exact="onClick">A</button>

    </details>

系统按键修饰符：https://cn.vuejs.org/guide/essentials/event-handling.html#system-modifier-keys

Vue 中的按键别名（.enter）和修饰键（.ctrl）及 组合键的使用： https://www.jianshu.com/p/818226fe813f

## 3. v-model
在表单输入元素或组件上创建双向绑定。
- 单个组件可以声明多个`v-model`
### 1.表单输入绑定

```

// v-model 在表单中是一个语法糖,与下面用法一致
<input :value="text" @input="event => text = event.target.value">
<input v-model="text">

// 在 textarea 和 select(绑定 value 和 change 事件) 中用法类似

```

### 修饰符
- .lazy: 默认情况下，`v-model`会在每次 `input` 事件后更新数据 (IME 拼字阶段的状态例外)。
- .number: 将用户输入自动转化为数字
- .trim: 移除用户输入内容中的两端空格

### 2. 父子组件数据双向绑定: <a href="#defineModel">defineModel()</a>
```Vue3
    // 3.4++ :推荐使用 defineModel()
    // defineModel 设置 default ，父组件必须设置初始值，不然数据会不同步
    <!-- Child.vue -->
    <script setup>
        const model = defineModel(); 
        // defineModel 返回值为 ref, .value 值同父组件一致， v-model 同步更新
        function update(){
            model.value++
        }
    </script>
    <template>
        <div>Parent bound v-model is: {{ model }}</div>
        <button @click="update">Increment</button>
    </template>
    <!-- Parent.vue -->
    <script setup>
        const countModel = ref(0)
    </script>
    <Child v-model="countModel></Child>
```

<details>
<summary>3.4 之前实现</summary>

```Vue3
    <!-- Child.vue -->
    <script setup>
        const props = defineProps(['modelValue'])
        const emit = defineEmits(['update:modelValue'])
    </script>

    <template>
        <input :value="props.modelValue" 
        @input="emit('update:modelValue',$event.target.value)">
    </template>

    <!-- Parent -->
    <scrpit setup>
        const foo = ref("Hello world!")
    </script>

    <Child 
    :modeValue="foo" 
    @update:modelValue="$event => (foo = $event)"></Child>
```

</details>

### 修饰符
- `.lazy` - 监听 change 事件而不是 input 
- `.number` - 将输入的合法字符串转为数字
- `.trim` - 移除输入内容两端空格

## 4. v-if
`v-if`指令用于条件性地渲染一块内容。这块内容只会在指令的表达式返回真值时才被渲染。

```
// v-if v-eles-if v-else
<div v-if="type === 'A'"> A </div>
<div v-else-if="type === 'B'"> B </div>
<div v-else-if="type === 'C'"> C </div>
<div v-else> Not A/B/C </div>
```
注意：`v-if`和`v-for`同时存在一个元素上时，`v-if`会被先执行.这两个不推荐同时使用。

## 5. v-show
另一个可以用来按条件显示一个元素的指令是 v-show。其用法与`v-if`基本一样：
- 不同之处在于`v-show`会在`DOM`渲染中保留该元素；`v-show`仅切换了该元素上名为`display`的 `CSS` 属性。
- `v-show` 不支持在 `<template>` 元素上使用，也不能和 `v-else` 搭配使用。

## 6. v-for
1. `v-for`指令基于一个数组来渲染一个列表。<br>
语法：`v-for="(item, index) in items"` 或 `v-for="item of items"`
```
// 例子：遍历并使用解构
<li v-for="{ message } in items">
  {{ message }}
</li>

<!-- 有 index 索引时 -->
<li v-for="({ message }, index) in items">
  {{ message }} {{ index }}
</li>
```
2. `v-for`遍历对象
```

// 例子：
const myObject = reactive({
  title: 'How to do lists in Vue',
  author: 'Jane Doe',
  publishedAt: '2016-04-10'
})

<li v-for="(value, key, index) in myObject"> // 若不写第二和三个参数，默认返回 value
  {{ index }}. {{ key }}: {{ value }}
</li>
<li v-for="(value, key) in myObject"> 
  {{ key }}: {{ value }}
</li>

// 可以用 of 代替 in 更接近 Javascript 用法
<li v-for="(value, key, index) of myObject"> // 若不写第二和三个参数，默认返回 value
  {{ index }}. {{ key }}: {{ value }}
</li>

```

注意：
- `v-if`和`v-for`：`v-if`会先执行，若两个在同一级上建议外层用`<template v-for="XXXX">`包裹
- `v-for`与`:key`

[v-for 与 v-if] https://cn.vuejs.org/guide/essentials/list.html#v-for-with-v-if

[展示过滤或排序后的结果] https://cn.vuejs.org/guide/essentials/list.html#displaying-filtered-sorted-results

## 7. v-slot
前置知识：<a href="#componentBase">组件基础</a>
`<slot>` 元素是一个插槽出口 (slot outlet)，标示了父元素提供的插槽内容 (slot content) 将在哪里被渲染。

- 插槽作用域可以访问到父元素
- 插槽可以给默认值，默认值将在没有传入内容时显示


```

// 这里有一个 <FancyButton> 组件(父组件)
<FancyButton>
  Click me! <!-- 插槽内容 -->
</FancyButton>

// 而 <FancyButton> 的模板是这样的：
<button class="fancy-btn">
  <slot></slot> <!-- 插槽出口 -->
</button>

// 渲染结果
<button class="fancy-btn">Click me!</button>
```
### 具名插槽
具名插槽指带有`name`的插槽。没有提供 `name` 的 `<slot>` 出口会隐式地命名为“default”。

```

<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>

// 父组件使用
<BaseLayout>
  <template v-slot:header>
    <!-- header 插槽的内容放这里 -->
  </template>
  // v-slot 有对应的简写 #
  <template #footer>
    <!-- footer 插槽内容 -->
  </template>
</BaseLayout>

```

### 条件插槽

```

// 与 v-if 一起使用
<template>
  <div class="card">
    <div v-if="$slots.header" class="card-header">
      <slot name="header" />
    </div>
  </div>
</template>

```

### 动态插槽

```

// 与 v-bind 动态名使用方法一致
<base-layout>

    <template v-slot:[dynamicSlotName]>
        ...
    </template>

    <!-- 缩写 -->
    <template #[dynamicSlotName]>
        ...
    </template>
</base-layout>


```

### 插槽内部向外部传参

- 具名插槽和默认插槽一起使用时，默认插槽在传入时最好用`<template>`包裹，避免`props`作用域困惑。 
- 作用域插槽在需要**同时封装逻辑**、**组合视图界面**时还是很有用

```

<!-- <MyComponent> 的模板 -->
<div>
  <slot :text="greetingMessage" :count="1"></slot>
</div>

<MyComponent v-slot="slotProps">
  {{ slotProps.text }} {{ slotProps.count }}
</MyComponent>

<!-- 具名插槽使用 -->
<!-- <MyComponent> 的模板 -->
<div>
  <slot name="header" :text="greetingMessage" :count="1"></slot>
</div>

<MyComponent #header="headerProps">
    {{ headerProps }}
</MyComponent>

// 解构获取参数
<MyComponent #header="{ text,count }">
    {{ headerProps }}
</MyComponent>
```

高级列表组件示例: https://cn.vuejs.org/guide/components/slots.html#fancy-list-example

## 自定义事件



# 计算属性
**计算属性值会基于其响应式依赖被缓存**,一个计算属性仅会在其响应式依赖更新时才重新计算。其还可以减少模板中的逻辑，提升可维护性。

```
// 例子
<template>
    <p>Has published books:</p>
    <span>{{ publishedBooksMessage }}</span>
</template>

<script>
import { reactive, computed } from 'vue'

const author = reactive({
  name: 'John Doe',
  books: [
    'Vue 2 - Advanced Guide',
    'Vue 3 - Basic Guide',
    'Vue 4 - The Mystery'
  ]
})

// 一个计算属性
const publishedBooksMessage - computed(() => {
    return author.book.length > 0 : 'Yes' : 'No'
})

</script>
```

注意：
- 可以通过`getter`和`setter`来创建"可写"的属性详情看文档
- **不要改变其他状态、在 getter 中做异步请求或者更改 DOM！**
- **避免直接修改计算属性值**

[Vue3 计算属性]：https://cn.vuejs.org/guide/essentials/computed.html


# Class 与 Style 绑定
## Class 绑定
方式1：对象式绑定
```Vue3
// 例子： 使用 v-bind 绑定 Class
<template>
    <div :class="classObejct"></div>
</template>

const isActive = ref(true);
const error = ref(null);

// 使用计算属性返回
const classObject = computed(() => ({
    active: isActive.value && !error.value,
    'text-danger': error.value && error.value.type === 'fatal'
}))

```

方式2：数组式绑定
```Vue3
const activeClass = ref('active')
const errorClass = ref('text-danger')

// 可以嵌套三元表达式使用
<div :class="[isActive ? activeClass : '', errorClass]"></div>
// 可以嵌套对象使用
<div :class="[{ [activeClass]: isActive }, errorClass]"></div>

```

组件中使用
```
// 单个根元素
// 父组件中添加给子组件添加class，会直接添加到子组件的元素中
// MyComponent.vue
<p class="foo bar">Hi1</p>

// 父组件
<MyComponent class='baz boo'></MyComponent>
// 渲染结果
<p class='foo bar baz boo'>Hi!</p>

// 多个根元素
// 需要通过 $attrs 来获取
<!-- MyComponent 模板使用 $attrs 时 -->
<p :class="$attrs.class">Hi!</p>
<span>This is a child component</span>

// 父组件
<MyComponent class="baz" />

// 渲染
<p class="baz">Hi!</p>
<span>This is a child component</span>
```

## Style 绑定
方式1： 对象绑定
```
const activeColor = ref('red')
const fontSize = ref(30)

<div :style="{color: activeColor,fontSize: fontSize+'px'}"></div>
// 推荐直接绑定一个对象
```

方式2：数组绑定
```
<div :style="[baseStyles, overridingStyles]"></div>
```

# 组件基础 <p id="componentBase"></p>
组件允许我们将 UI 划分为独立的、可重用的部分，并且可以对每个部分进行单独的思考。
- 组件需要转换为相应等价的`kebab-case` (短横线连字符)形式
- 没有使用`setup`需要通过`componetns`来注册组件(使用`PascalCase`命名)

## 单文件组件(SFC)

```
// ButtonCounter.vue
<script setup>
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <button @click="count++">You clicked me {{ count }} times.</button>
</template>

// 使用
<script setup>
import ButtonCounter from './ButtonCounter.vue'
</script>

<template>
  <h1>Here is a child component!</h1>
  <ButtonCounter />
</template>
```

## 动态组件
有些场景会需要在两个组件间来回切换，比如 Tab 界面：


```
<!-- currentTab 改变时组件也改变 -->
<component :is="tabs[currentTab]"></component>

<!-- 被传给 :is 的值可以是以下几种： -->
<!-- 被注册的组件名 -->
<!-- 导入的组件对象 -->
```

## defineProps() 和 defineEmits()
- defineProps 和 defineEmits 都是只能在`<script setup>`中使用的编译器宏。他们不需要导入，且会随着`<script setup>`的处理过程一同被编译掉。
- defineProps 接收与 props 选项相同的值，defineEmits 接收与 emits 选项相同的值。
- defineProps 和 defineEmits 在选项传入后，会提供恰当的类型推导。
- 传入到 defineProps 和 defineEmits 的选项会从 setup 中提升到模块的作用域。因此，传入的选项不能引用在 setup 作用域中声明的局部变量。这样做会引起编译错误。但是，它可以引用导入的绑定，因为它们也在模块作用域内。(上级组件无法通过defineProps使用子组件的参数)
- 对于过长名称使用`camelCase`方式命名
- 使用`v-bind`绑定对象，传递多个<a href="#moreProps">`prop`</a>
- 单向绑定，props随父组件更新而变化，而引用数据类型(Array,Object)可以更改(最好通过emit返回)

```
// defineProps使用
<!-- BlogPost.vue -->
<script setup>
defineProps(['title'])
</script>

<template>
  <h4>{{ title }}</h4>
</template>

// 父组件
<BlogPost title="Why Vue is so fun" />



// defineEmits使用
<BlogPost
  ...
  @enlarge-text="postFontSize += 0.1"
 />

<template>
  <div class="blog-post">
    <h4>{{ title }}</h4>
    <!-- template 中直接调用 $emit -->
    <button @click="$emit('enlarge-text')">Enlarge text</button>
  </div>
</template>

<script setup>
// setup中使用 emit
const emit = defineEmits(['enlarge-text'])
emit('enlarge-text'); // 这样会报错
</script>

// 不使用<script setup>,则从setup(props, ctx),ctx中获取emit
export default {
    emits: ['inFocus', 'submit'],
    setup(props,ctx){
        ctx.emit('submit)
    }
}
```

[defineProps TS 声明] https://cn.vuejs.org/api/sfc-script-setup.html#type-only-props-emit-declarations
<h2 id="defineModel">2. defineModel (3.4+)</h2>
这个宏可以用来声明一个双向绑定 prop，通过父组件的 v-model 来使用。

## 使用: defineModel(propsName?, {options?...})
    propsName: 默认为 modelValue,修改时触发 update:modelValue。

    // 声明 "count" prop，由父组件通过 v-model:count 使用
    // 修改时会触发 update:count
    const count = defineModel("count")


# 依赖注入
`provide` 和 `inject` 可以帮助我们解决**深层**传递问题。一个父组件相对于其所有的后代组件，会作为依赖提供者。

## provide(提供)

- `provide() `函数接收两个参数。第一个参数被称为注入名，可以是一个字符串或是一个 Symbol。后代组件会用注入名来查找期望注入的值。一个组件可以多次调用 provide()，使用不同的注入名，注入不同的依赖值。
- 第二个参数是提供的值，值可以是任意类型，包括响应式的状态，比如一个 ref。
- **建议尽可能将任何对响应式状态的变更都保持在供给方组件中**
- 大型应用最好使用`Symbol`作为注入名

```

<script setup>
    import { provide } from 'vue'

    provide(/* 注入名 */ 'message', /* 值 */ 'hello!')

    // 例子：
    import { ref, provide } from 'vue'

    const count = ref(0)
    provide('key', count)
</script>

```

## Inject (注入)​
- 第一个参数为`provide`的注入名。
- 第二参数设置默认值(可以使用工厂函数来创建默认值)。
- 第三个参数表示默认值应当被当成一个工厂函数。

```

<script setup>
import { inject } from 'vue'

const message = inject('message','这是默认值')
</script>

```


# 异步组件
在大型项目中，我们可能需要拆分应用为更小的块，并仅在需要时再从服务器加载相关组件。Vue 提供了 `defineAsyncComponent` 方法来实现此功能
- 

```

import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() => {
  return new Promise((resolve, reject) => {
    // ...从服务器获取组件
    resolve(/* 获取到的组件 */)
  })
})
// ... 像使用其他一般组件一样使用 `AsyncComp`

<!-- 参数 -->
const AsyncComp = defineAsyncComponent({
  // 加载函数
  loader: () => import('./Foo.vue'),

  // 加载异步组件时使用的组件
  loadingComponent: LoadingComponent,
  // 展示加载组件前的延迟时间，默认为 200ms
  delay: 200,

  // 加载失败后展示的组件
  errorComponent: ErrorComponent,
  // 如果提供了一个 timeout 时间限制，并超时了
  // 也会显示这里配置的报错组件，默认值是：Infinity
  timeout: 3000
})

 <!-- 例子 -->
const AsyncComp = defineAsyncComponent(() =>
  import('./components/MyComponent.vue')
)
<!-- 最后得到的 AsyncComp 是一个外层包装过的组件，仅在页面需要它渲染时才会调用加载内部实际组件的函数。它会将接收到的 props 和插槽传给内部组件，所以你可以使用这个异步的包装组件无缝地替换原始组件，同时实现延迟加载。 -->

```

# 生命周期

- beforeCreate
- created
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeUmmount
- ummounted

## 注册周期钩子
常用：onMounted、onUpdated 和 onUnmounted。

### onMounted
注册一个回调函数，在组件挂载完成后执行。

组件在以下情况下被视为已挂载：
- 其所有同步子组件都已经被挂载 (不包含异步组件或 <Suspense> 树内的组件)。
- 其自身的 DOM 树已经创建完成并插入了父容器中。注意仅当根容器在文档中时，才可以保证组件 DOM 树也在文档中。
- 在服务器渲染期间不会被调用

### onUpdated()
注册一个回调函数，在组件因为响应式状态变更而更新其 DOM 树之后调用。

- 父组件的更新钩子将在其子组件的更新钩子之后调用。
- 这个钩子会在组件的任意 DOM 更新后被调用，这些更新可能是由不同的状态变更导致的，因为多个状态变更可以在同一个渲染周期中批量执行 (考虑到性能因素)。如果你需要在某个特定的状态更改后访问更新后的 DOM，请使用 `nextTick()` 作为替代。
- 在服务器渲染期间不会被调用

### onUnmounted() 
注册一个回调函数，在组件实例被卸载之后调用。

一个组件在以下情况下被视为已卸载：
- 其所有子组件都已经被卸载。
- 所有相关的响应式作用 (渲染作用以及 setup() 时创建的计算属性和侦听器) 都已经停止。
- 这个钩子在服务器端渲染期间不会被调用。

可以在这个钩子中手动清理一些副作用，例如计时器、DOM 事件监听器或者与服务器的连接。

### onBeforeMount()
注册一个钩子，在组件被挂载之前被调用。
当这个钩子被调用时，组件已经完成了其响应式状态的设置，但还没有创建 DOM 节点。它即将首次执行 DOM 渲染过程。
这个钩子在服务器端渲染期间不会被调用。

### onBeforeUpdate()
注册一个钩子，在组件即将因为响应式状态变更而更新其 DOM 树之前调用。

这个钩子可以用来在 Vue 更新 DOM 之前访问 DOM 状态。在这个钩子中更改状态也是安全的。
这个钩子在服务器端渲染期间不会被调用。

### onBeforeUnmount()
注册一个钩子，在组件实例被卸载之前调用。

当这个钩子被调用时，组件实例依然还保有全部的功能。
这个钩子在服务器端渲染期间不会被调用。

### onErrorCaptured()
注册一个钩子，在捕获了后代组件传递的错误时调用。

错误可以从以下几个来源中捕获：
组件渲染、事件处理器、生命周期钩子、setup() 函数、侦听器、自定义指令钩子、过渡钩子

这个钩子带有三个实参：错误对象、触发该错误的组件实例，以及一个说明错误来源类型的信息字符串。
这个钩子仅在开发模式下可用，且在服务器端渲染期间不会被调用。

### onRenderTracked()
注册一个调试钩子，当组件渲染过程中追踪到响应式依赖时调用。
这个钩子仅在开发模式下可用，且在服务器端渲染期间不会被调用。

### onRenderTriggered()
注册一个调试钩子，当响应式依赖的变更触发了组件渲染时调用。
这个钩子仅在开发模式下可用，且在服务器端渲染期间不会被调用。

### onActivated()
注册一个回调函数，若组件实例是 <KeepAlive> 缓存树的一部分，当组件被插入到 DOM 中时调用。

### onDeactivated()
注册一个回调函数，若组件实例是 <KeepAlive> 缓存树的一部分，当组件从 DOM 中被移除时调用。

### onServerPrefetch()
注册一个异步函数，在组件实例在服务器上被渲染之前调用。

如果这个钩子返回了一个 Promise，服务端渲染会在渲染该组件前等待该 Promise 完成。
这个钩子仅会在服务端渲染中执行，可以用于执行一些仅存在于服务端的数据抓取过程。

组合式 API：生命周期钩子  https://cn.vuejs.org/api/composition-api-lifecycle.html#composition-api-lifecycle-hooks


# 组合式函数
在 Vue 应用的概念中，“组合式函数”(Composables) 是一个利用 Vue 的组合式 API 来封装和复用有**状态逻辑**的函数。

- 传入响应式参数`ref`,可以通过`toValue`和`watchEffect`值改变后函数再次执行。
- 组合式函数约定用驼峰命名法命名，并以“use”作为开头。
- 输入参数即便不依赖于 ref 或 getter 的响应性，组合式函数也可以接收它们作为参数。如果你正在编写一个可能被其他开发者使用的组合式函数，最好处理一下输入参数是 ref 或 getter 而非原始值的情况。可以利用 toValue() 工具函数来实现：

```

// mouse.js
import { ref, onMounted, onUnmounted } from 'vue'

// 按照惯例，组合式函数名以“use”开头
export function useMouse() {
  // 被组合式函数封装和管理的状态
  const x = ref(0)
  const y = ref(0)

  // 组合式函数可以随时更改其状态。
  function update(event) {
    x.value = event.pageX
    y.value = event.pageY
  }

  // 一个组合式函数也可以挂靠在所属组件的生命周期上
  // 来启动和卸载副作用
  onMounted(() => window.addEventListener('mousemove', update))
  onUnmounted(() => window.removeEventListener('mousemove', update))

  // 通过返回值暴露所管理的状态
  return { x, y }
}

// 组件中使用
<script setup>
import { useMouse } from './mouse.js'

const { x, y } = useMouse()
</script>

<template>Mouse position is at: {{ x }}, {{ y }}</template>

```


# 文档
[Vue3 官方文档] https://cn.vuejs.org/guide/essentials/template-syntax.html
