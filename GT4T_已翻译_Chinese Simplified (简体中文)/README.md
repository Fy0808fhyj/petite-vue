# 小视野

 `petite-vue` 是[Vue](https://vuejs.org)的替代发行版，针对[渐进增强](https://developer.mozilla.org/en-US/docs/Glossary/Progressive_Enhancement)进行了优化。它提供与标准 Vue.js 相同的模板语法和反应性心智模型。但是，它专门针对在服务器框架呈现的现有 HTML 页面上“散布”少量交互进行了优化。在[它与标准 Vue 有何不同](#comparison-with-standard-vue)上查看更多详细信息。

- 只有 ~6k
- Vue 兼容的模板语法
- 基于 DOM，就地改变
- 由 `@vue/reactivity` 驱动

## 地位

- 这是很新的。可能存在错误并且可能仍然存在 API 更改，所以**使用风险自负。**它是否可用？非常。查看[examples](https://github.com/vuejs/petite-vue/tree/main/examples)以了解它的功能。

- 问题列表是有意禁用的，因为我现在有更高优先级的事情要关注并且不想分心。如果发现错误，则必须解决它或提交 PR 自行修复。也就是说，请随时使用讨论选项卡互相帮助。

- 目前不太可能接受功能请求 - 该项目的范围有意保持在最低限度。

## 用法

 `petite-vue` 可以在没有构建步骤的情况下使用。只需从 CDN 加载它：


```html
<script src="https://unpkg.com/petite-vue" defer init></script>

<!-- anywhere on the page -->
<div v-scope="{ count: 0 }">
  {{ count }}
  <button @click="count++">inc</button>
</div>
```

- 使用 `v-scope` 标记页面上应由 `petite-vue` 控制的区域。
-  `defer` 属性使脚本在解析 HTML 内容后执行。
-  `init` 属性告诉 `petite-vue` 自动查询和初始化页面上所有具有 `v-scope` 的元素。

### 手动初始化

如果你不想要自动初始化，请删除 `init` 属性并将脚本移动到 `<body>` 的末尾：


```html
<script src="https://unpkg.com/petite-vue"></script>
<script>
  PetiteVue.createApp().mount()
</script>
```

或者，使用 ES 模块构建：


```html
<script type="module">
  import { createApp } from 'https://unpkg.com/petite-vue?module'
  createApp().mount()
</script>
```

### 生产 CDN URL

简短的 CDN URL 用于原型设计。对于生产使用，请使用完全解析的 CDN URL 以避免解析和重定向成本：

- 全局构建： `https://unpkg.com/petite-vue@0.2.2/dist/petite-vue.iife.js`
  - 公开 `PetiteVue` 全局，支持自动初始化
- ESM 构建： `https://unpkg.com/petite-vue@0.2.2/dist/petite-vue.es.js`
  - 必须与 `<script type="module">` 一起使用

### 根范围

 `createApp` 函数接受一个数据对象作为所有表达式的根作用域。这可用于引导简单的一次性应用程序：


```html
<script type="module">
  import { createApp } from 'https://unpkg.com/petite-vue?module'

  createApp({
    // exposed to all expressions
    count: 0,
    // getters
    get plusOne() {
      return this.count + 1
    },
    // methods
    increment() {
      this.count++
    }
  }).mount()
</script>

<!-- v-scope value can be omitted -->
<div v-scope>
  <p>{{ count }}</p>
  <p>{{ plusOne }}</p>
  <button @click="increment">increment</button>
</div>
```

注意 `v-scope` 在这里不需要有值，只是作为提示 `petite-vue` 处理元素。

### 显式挂载目标

你可以指定挂载目标（选择器或元素）以将 `petite-vue` 限制为仅在页面的该区域：


```js
createApp().mount('#only-this-div')
```

这也意味着你可以有多个 `petite-vue` 应用程序来控制同一页面上的不同区域：


```js
createApp({
  // root scope for app one
}).mount('#app1')

createApp({
  // root scope for app two
}).mount('#app2')
```

### 生命周期事件

你可以监听每个元素的特殊 `vue:mounted` 和 `vue:unmounted` 生命周期事件（自 v0.4.0 起需要 `vue:` 前缀）：


```html
<div
  v-if="show"
  @vue:mounted="console.log('mounted on: ', $el)"
  @vue:unmounted="console.log('unmounted: ', $el)"
></div>
```

### `v-效果 `

使用 `v-effect` 执行**被动的**内联语句：


```html
<div v-scope="{ count: 0 }">
  <div v-effect="$el.textContent = count"></div>
  <button @click="count++">++</button>
</div>
```

该效果使用 `count` 这是一个反应式数据源，因此它会在 `count` 更改时重新运行。

替换原始 Vue TodoMVC 示例中的 `todo-focus` 指令的另一个示例：


```html
<input v-effect="if (todo === editedTodo) $el.focus()" />
```

### 成分

“组件”的概念在 `petite-vue` 中有所不同，因为它更简单。

首先，可以使用函数创建可重用的作用域逻辑：


```html
<script type="module">
  import { createApp } from 'https://unpkg.com/petite-vue?module'

  function Counter(props) {
    return {
      count: props.initialCount,
      inc() {
        this.count++
      },
      mounted() {
        console.log(`I'm mounted!`)
      }
    }
  }

  createApp({
    Counter
  }).mount()
</script>

<div v-scope="Counter({ initialCount: 1 })" @vue:mounted="mounted">
  <p>{{ count }}</p>
  <button @click="inc">increment</button>
</div>

<div v-scope="Counter({ initialCount: 2 })">
  <p>{{ count }}</p>
  <button @click="inc">increment</button>
</div>
```

### 带模板的组件

如果你还想重复使用一个模板，你可以在作用域对象上提供一个特殊的 `$template` 键。该值可以是模板字符串，也可以是 `<template>` 元素的 ID 选择器：


```html
<script type="module">
  import { createApp } from 'https://unpkg.com/petite-vue?module'

  function Counter(props) {
    return {
      $template: '#counter-template',
      count: props.initialCount,
      inc() {
        this.count++
      }
    }
  }

  createApp({
    Counter
  }).mount()
</script>

<template id="counter-template">
  My count is {{ count }}
  <button @click="inc">++</button>
</template>

<!-- reuse it -->
<div v-scope="Counter({ initialCount: 1 })"></div>
<div v-scope="Counter({ initialCount: 2 })"></div>
```

推荐使用 `<template>` 方法而不是内联字符串，因为从本机模板元素克隆更有效。

### 全局状态管理

你可以使用 `reactive` 方法（从 `@vue/reactivity` 重新导出）来创建全局状态单例：


```html
<script type="module">
  import { createApp, reactive } from 'https://unpkg.com/petite-vue?module'

  const store = reactive({
    count: 0,
    inc() {
      this.count++
    }
  })

  // manipulate it here
  store.inc()

  createApp({
    // share it with app scopes
    store
  }).mount()
</script>

<div v-scope="{ localCount: 0 }">
  <p>Global {{ store.count }}</p>
  <button @click="store.inc">increment</button>

  <p>Local {{ localCount }}</p>
  <button @click="localCount++">increment</button>
</div>
```

### 自定义指令

还支持自定义指令，但具有不同的界面：


```js
const myDirective = (ctx) => {
  // the element the directive is on
  ctx.el
  // the raw value expression
  // e.g. v-my-dir="x" then this would be "x"
  ctx.exp
  // v-my-dir:foo -> "foo"
  ctx.arg
  // v-my-dir.mod -> { mod: true }
  ctx.modifiers
  // evaluate the expression and get its value
  ctx.get()
  // evaluate arbitrary expression in current scope
  ctx.get(`${ctx.exp} + 10`)

  // run reactive effect
  ctx.effect(() => {
    // this will re-run every time the get() value changes
    console.log(ctx.get())
  })

  return () => {
    // cleanup if the element is unmounted
  }
}

// register the directive
createApp().directive('my-dir', myDirective).mount()
```

 `v-html` 是这样实现的：


```js
const html = ({ el, get, effect }) => {
  effect(() => {
    el.innerHTML = get()
  })
}
```

### 自定义分隔符 (0.3+)

你可以通过将 `$delimiters` 传递到你的根范围来使用自定义分隔符。这在与也使用 mustaches 的服务器端模板语言一起工作时很有用：


```js
createApp({
  $delimiters: ['${', '}']
}).mount()
```

## 例子

查看[示例目录](https://github.com/vuejs/petite-vue/tree/main/examples)。

## 特征

### 仅 `petite-vue`

- `v-scope`
- `v-效果 `
-  `@vue:mounted` & `@vue:unmounted` 事件

### 有不同的行为

- 在表达式中， `$el` 指向指令绑定到的当前元素（而不是组件根元素）
-  `createApp()` 接受全局状态而不是组件
- 组件被简化为返回对象的函数
- 自定义指令具有不同的界面

### 兼容视图

-  `{{}}` 文本绑定（可配置自定义分隔符）
-  `v-bind`（包括 `:` 速记和类/样式特殊处理）
-  `v-on`（包括 `@` 速记和所有修饰符）
-  `v-model`（所有输入类型 + 非字符串 `:value` 绑定）
-  `v-if`/ `v-else`/ `v-else-if`
- `v-for`
- `v-show`
- `v-html`
- `v-文本 `
- `in-for'
- `v-一次 `
- `v-斗篷 `
- ` 反应性（）`
- `nextTick()`
- 模板引用

### 不支持

一些特征被删除是因为它们在渐进增强的背景下具有相对较低的效用/大小比。如果你需要这些功能，你应该只使用标准的 Vue。

-  `ref()`, `computed()` 等。
- 渲染函数（ `petite-vue` 没有虚拟 DOM）
- 集合类型的反应性（地图、集合等，为较小的尺寸而移除）
- 过渡、KeepAlive、传送、悬念
-  `v-for` 深度解构
- `v-on="对象"`
-  `v-is` & `<component:is="xxx">`
-  `v-bind:style` 自动前缀

## 与标准 Vue 的比较

 `petite-vue` 的意义不仅仅在于体积小。它是关于为预期用例使用最佳实现（渐进式增强）。

标准 Vue 可以在有或没有构建步骤的情况下使用。当使用构建设置（例如使用单文件组件）时，我们会预编译所有模板，因此无需在运行时进行模板处理。多亏了 tree-shaking，我们可以在标准 Vue 中发布可选功能，这些功能在不使用时不会膨胀你的包大小。这是标准 Vue 的最佳用法，但由于它涉及构建设置，因此更适合构建 SPA 或交互相对较多的应用程序。

在没有构建步骤并安装到 in-DOM 模板的情况下使用标准 Vue 时，它​​不是最佳选择，因为：

- 我们必须将 Vue 模板编译器发送到浏览器（13kb 额外大小）
- 编译器必须从已经实例化的 DOM 中检索模板字符串
- 然后编译器将字符串编译成 JavaScript 渲染函数
- Vue 然后用渲染函数生成的新 DOM 替换现有的 DOM 模板。

 `petite-vue` 通过遍历现有的 DOM 并将细粒度的反应效果直接附加到元素来避免所有这些开销。DOM _是_模板。这意味着 `petite-vue` 在渐进增强场景中效率更高。

这也是 Vue 1 的工作方式。这里的权衡是这种方法与 DOM 耦合，因此不适合与平台无关的渲染或 JavaScript SSR。我们也失去了使用渲染函数进行高级抽象的能力。然而，正如你可能知道的那样，在渐进增强的环境中很少需要这些功能。

## 与高山的比较

 `petite-vue` 确实解决了与[Alpine](https://alpinejs.dev)类似的范围，但目标是 (1) 更小和 (2) 更兼容 Vue。

-  `petite-vue` 大约是 Alpine 大小的一半。

-  `petite-vue` 没有过渡系统（也许这可以是一个可选插件）。

- 尽管 Alpine 在很大程度上类似于 Vue 的设计，但在许多情况下其行为与 Vue 本身不同。它也可能在未来与 Vue 产生更多分歧。这很好，因为 Alpine 不必将其设计限制为严格遵循 Vue——​​它应该可以自由地朝着对其目标有意义的方向发展。

  相比之下， `petite-vue` 将尽可能尝试与标准 Vue 行为保持一致，以便在需要时减少迁移到标准 Vue 的摩擦。它旨在成为**Vue 生态系统的一部分**以涵盖渐进式增强用例，其中标准 Vue 现在的优化程度较低。

## 安全和 CSP

 `petite-vue` 评估模板中的 JavaScript 表达式。这意味着**如果** `petite-vue` 被挂载在包含来自用户数据的未净化 HTML 的 DOM 区域，它可能会导致 XSS 攻击。**如果你的页面呈现用户提交的 HTML，你应该更喜欢使用[显式挂载目标](#explicit-mount-target)初始化 `petite-vue` 以便它只处理受控的部分由你**。你还可以为 `v-scope` 属性清理任何用户提交的 HTML。

 `petite-vue` 使用 `new Function()` 计算表达式，这在严格的 CSP 设置中可能被禁止。没有提供 CSP 构建的计划，因为它涉及运送一个表达式解析器，这违背了轻量级的目的。如果你有严格的 CSP 要求，你可能应该使用标准 Vue 并预编译模板。

## 执照

和
