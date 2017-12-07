+++
title = "快速入门 - Vue2 Tutorials (一)"
description = "Vue2 Tutorials 01 Quick Start"
date = 2017-07-06T14:53:34+08:00
tags = ["vue", "前端"]
categories = ["vue", "前端"]
draft = false
+++


Vue 的[官方文档](https://cn.vuejs.org/v2/guide/) 对 Vue 介绍非常详细，但官方文档使用在 HTML 中引入 vue 的方式进行讲解，而实际项目中一般使用脚手架如 vue-cli 初始化项目。以至于刚看完文档时，却依旧不能立即立即 vue-cli 创建的项目代码。所以本文 vue-cli 构建的项目为基础，详细解释其代码及对应的概念，并进行简单的实践。

> 本文的代码在 [https://github.com/nodejh/vue2-tutorials/tree/master/01.QuickStart](https://github.com/nodejh/vue2-tutorials/tree/master/01.QuickStart)

## 命令行工具

### 安装 vue-cli 并初始化项目

首先要全局安装 vue-cl：

```shell
$ npm install --global vue-cli
```

然后使用 vue-cli 初始化一个基于 webpack 模板的新项目，除了 `Install vue-router?` 选 `N (No)`，其余都可以直接回车选 `Y (Yes)`，因为我们暂时不会讲到 `vue-router`：

```shell
$ vue init webpack demo
? Project name demo
? Project description A Vue.js project

? Author nodejh <jianghangscu@gmail.com>
? Vue build standalone
? Install vue-router? No
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Setup unit tests with Karma + Mocha? Yes
? Setup e2e tests with Nightwatch? Yes
```

### 安装依赖

```shell
$ cd demo
$ npm install
$ npm run dev
```

然后打开浏览器，输入 `http://localhost:8080` 就能看到界面。

接下来分析一下代码。

## 代码分析

项目目录结构如下：

```shell
├── README.md
├── build    # 编译项目的配置文件目录
├── config    # 配置文件目录
├── src    # 项目主要代码目录
├── static    # 静态资源
├── test    # 测试文件目录
```

开发阶段的主要代码都在 `src` 目录中编写，vue-cli 默认生成了一些代码：

```shell
src
├── App.vue
├── assets
│   └── logo.png
├── components
│   └── Hello.vue
└── main.js
```

可以发现，代码的后缀名有两种：

- `.js` JS 文件
- `.vue` Vue 组件，里面定义了 Vue 实例、模板、样式等。需要由 webpack 等工具来转换为 js 代码

接下来会逐一解释这些文件及代码。

### main.js

`main.js` 是项目的入口文件，也是 webpack 打包的入口文件。里面最代码很少，主要就是通过 `new Vue()` 创建 Vue 实例：

```js
new Vue({
  el: '#app',
  template: '<App/>',
  components: { App },
});
```

每个 Vue.js 应用都是通过构造函数 Vue 创建一个 Vue 的根实例启动的。

在实例化 Vue 时，传入一个了一个对象，对象包含以下几个选项。

#### el

`el` 的值是 Vue 实例的挂载目标，这里是 `#app`，也就是 `demo/index.html` 中 `id="app"` 这个元素：

```html
<div id="app"></div>
```

`el` 必须是一个已存在的元素。

[api/#el](https://cn.vuejs.org/v2/api/#el)


#### components

在说 `template` 之前，先来看看 `components` 属性。

`components: { App }` 等价于 `components: { App: App }`，是一个包含了对 Vue 实例可见的组件的哈希表。只有在 `components` 里面列出来的组件，才可以在 `template` 里面使用。

如果我们把 `components: { App }` 改为 `components: { App }` 改为 `components: { MyApp: App }`，那么在 `template` 里面就需要这样使用：`template: '<my-app />'`。

由于 HTML 标签不区分大小写，所以 `components` 里面的驼峰命名会自动转换为短横线。详见 [camelCase vs. kebab-case](https://cn.vuejs.org/v2/guide/components.html#camelCase-vs-kebab-case)


#### template

`template` 就是挂载到页面的模板。

这里的值是 `<App/>` 组件就是 `components` 属性中的 `App`，也就是通过 `import` 引入的 `App` 这个模板。

```js
new Vue({
  el: '#app',
  // 这里的 <App/> 就是 components 属性的值 App
  template: '<App/>',
  components: { App },
});
```

所以这段代码的含义就是，将 `<App/>` 这个模板挂载到元素 `#app` 上。

### src/App.vue

`src/App.vue` 是一个典型的[单文件组件](https://cn.vuejs.org/v2/guide/single-file-components.html)。实际在项目中，我们写的基本都是组件，再根据需要用组件组成页面，这其实就是组件化。组件与组件之间相互独立，项目结构更加清晰，也更有利于维护。

一个组件里面封装了 HTML、CSS 和 JS，有自己独立的样式和逻辑。

`<template>` 就是组件中的模板，模板的代码都在 `<template>` 标签中，除 `<hello>` 之外都是普通的 HTML。因为 `hello` 也是一个组件，然后通过标签的形式注入到模板中。

为什么模板中能使用 `hello` 这个组件呢？

这是因为 `<script></script>` 标签里面定义了 `Hello`（首字母大写）这个组件：

```js
import Hello from './components/Hello'

export default {
  name: 'app',
  components: {
    // Hello 组件，即 ./components/Hello 的一个引用
    Hello  
  }
}
```

这里 `components` 属性的含义，在之前已经提到过了，只有在 `components` 里面列出来的组件，才能被模板使用。这里列出了 `Hello` 这个组件，所以在 `<template>` 中我们可以使用 `<hello>`（前面也提到过， vue 会自动将驼峰法命名转为短横线）。

而 `components` 属性里面的 `Hello`，则是 `./components/Hello` 这个组件的一个引用：

```js
import Hello from './components/Hello'
```

最后就是 `<style>` 标签，里面就是普通的 CSS 了。


### src/components/Hello.vue

最后再来看看 `src/components/Hello.vue` 这个组件的代码。

基本跟 `src/App.vue` 是一样的，除了下面这两个地方之外：

```html
<h1>{{ msg }}</h1>
```

```js
data () {
  return {
    msg: 'Welcome to Your Vue.js App'
  }
}
```

恭喜你！看到这里，我们就可以真正开始写代码了。

`{{}}` 是 Vue 的一个模板语法，文本插值。如上面的例子所示，我们在 `data` 里面定义一个对象，就可以在模板中通过 `{{ }}` 来访问。

`data` 虽然是一个函数，但它执行之后就等价于：

```js
data: {
  msg: 'Welcome to Your Vue.js App'
}
```

当我们改变 `msg` 的值，在页面上渲染出来的数据也会改变。也就是数据和 DOM 绑定在了一起。


## 模板语法

### 插值

#### 文本插值

上面我们已经接触到了文本插值 `{{}}`，`{{ msg }}` 将会被替代为对应数据对象上 msg 属性的值。无论何时，绑定的数据对象（即 `data`）上 msg 属性发生了改变，插值处的内容都会更新。

通过使用 v-once 指令，我们也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上所有的数据绑定：

```html
<h1 v-once>This will never change: {{ msg }}</h1>
```

#### 纯 HTML

双大括号会将数据解释为纯文本，而非 HTML 。为了输出真正的 HTML ，需要使用 `v-html` 指令：

```html
<div v-html="rawHtml"></div>
```

这个 div 的内容将会被替换成为属性值 rawHtml，直接作为 HTML —— 数据绑定会被忽略。注意，你不能使用 v-html 来复合局部模板，因为 Vue 不是基于字符串的模板引擎。组件更适合担任 UI 重用与复合的基本单元。

> 你的站点上动态渲染的任意 HTML 可能会非常危险，因为它很容易导致 XSS 攻击。请只对可信内容使用 HTML 插值，绝不要对用户提供的内容插值。

#### JS 表达式

`{{}}` 中也可以写 JS 表达式：

```js
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

### 指令

指令（Directives）是带有 `v-` 前缀的特殊属性。

#### v-bind

`{{}}` 不能在 HTML 属性中使用。针对 HTML 属性需要使用 `v-bind`：

```html
<div v-bind:id="dynamicId"></div>
```

这对布尔值的属性也有效 —— 如果条件被求值为 `false` 的话该属性会被移除：

```html
<button v-bind:disabled="isButtonDisabled">Button</button>
```

`v-bind` 也可以缩写：

```html
<div :id="dynamicId"></div>
<button :disabled="isButtonDisabled">Button</button>
```

#### v-on

`v-on` 用来监听 DOM 事件：

```html
<button v-on:click="doSomething"></button>
```

也可以缩写成下面这样：

```html
<button @click="doSomething"></button>
```

#### v-if

```html
<template>
  <p v-if="seen">Now you see me</p>
</template>

<script>
export default {
  name: 'hello',
  data: {
    seen: true
  }
}
</script>
```


这里 `v-if` 指令将根据表达式 `seen` 的值的真假来移除/插入 `<p>` 元素。


#### v-for

`v-for` 指令可以绑定数组的数据来渲染一个项目列表：


```html
<template>
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</template>

<script>
  export default {
    data: {
      todos: [
        { text: '学习 JavaScript' },
        { text: '学习 Vue' },
        { text: '整个牛项目' }
      ]
    }
  }
</script>
```

## 实践

让我们把目光回到 `Hello.vue`。在这个组件里面有一些链接列表， Essential Links 和 Ecosystem，这些列表直接使用 HTML 编写：

```html
<ul>
  <li><a href="https://vuejs.org" target="_blank">Core Docs</a></li>
  <li><a href="https://forum.vuejs.org" target="_blank">Forum</a></li>
  <li><a href="https://gitter.im/vuejs/vue" target="_blank">Gitter Chat</a></li>
  <li><a href="https://twitter.com/vuejs" target="_blank">Twitter</a></li>
  <br>
  <li><a href="http://vuejs-templates.github.io/webpack/" target="_blank">Docs for This Template</a></li>
</ul>
<h2>Ecosystem</h2>
<ul>
  <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
  <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
  <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
  <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
</ul>
```

按照传统的写法，如果我们需要往里面添加链接的时候，每次我们都得添加 `<li>` 和 `<a>` 标签。思考两个问题：

- 添加几个链接还好，如果要添加非常非常多呢？难到要复制几十次 `<li>` 和 `<a>` 标签？
- 如果要动态改变链接列表呢？难道要使用 `innerHTML` 等方法修改 DOM？

聪明的你可能已经想到了，很明显不需要这么做，我们可以使用模板语法。将链接信息写到 Vue 的数据对象 `data` 里面，然后通过动态绑定的方式，将数据绑定到 DOM。

所以修改如下：

```html
<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h2>Essential Links</h2>
    <ul>
      <li v-for="essentialLink in essentialLinks">
        <a :href="essentialLink.link" target="_blank">{{ essentialLink.text }}</a>
      </li>
      <br>
      <li><a href="http://vuejs-templates.github.io/webpack/" target="_blank">Docs for This Template</a></li>
    </ul>
    <h2>Ecosystem</h2>
    <ul>
      <li v-for="ecosystem in ecosystems">
        <a :href="ecosystem.link" target="_blank">{{ ecosystem.text }}</a>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      ecosystems: [
        {
          link: 'http://router.vuejs.org/',
          text: 'vue-router'
        },
        {
          link: 'http://vuex.vuejs.org/',
          text: 'vuex'
        },
        {
          link: 'http://vue-loader.vuejs.org/',
          text: 'vue-loader'
        },
        {
          link: 'https://github.com/vuejs/awesome-vue',
          text: 'awesome-vue'
        }
      ],
      essentialLinks: [
        {
          link: 'https://vuejs.org',
          text: 'Core Docs'
        },
        {
          link: 'https://forum.vuejs.org',
          text: 'Forum'
        },
        {
          link: 'https://gitter.im/vuejs/vue',
          text: 'Gitter Chat'
        },
        {
          link: 'https://github.com/vuejs/awesome-vue',
          text: 'awesome-vue'
        },
        {
          link: 'https://twitter.com/vuejs',
          text: 'Twitter'
        }
      ]
    }
  }
}
</script>
```

这样我们就把数据和视图分开了，模板里面的代码也简洁了很多，不再需要写很多重复的代码。并且根据不同数据，我们也能展示出不同的 UI。

## 总结

本文详细讲解了 vue-cli 初始化的项目代码，并且在讲解代码的过程中，介绍了构造 vue 对象的一些参数，以及 vue 的一些基本概念，比如模板语法中的插值和指令。最后通过修改代码对以上知识点进行实践。

相信看到了这里，你对如何使用 vue 写一个项目已经有了初步了解。当然，看完本文，可能还有很多概念理解不清楚，这时推荐去看一下 vue 的[官方文档](https://cn.vuejs.org/)，这个时候再去看[官方文档](https://cn.vuejs.org/)，应该就会轻松很多了。
