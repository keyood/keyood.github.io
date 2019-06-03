---
layout:     post
title:      "nuxt 进阶"
subtitle:   "主要介绍各个模块"
date:       2018-09-06
author:     "keyood"
header-img: "img/post-bg-js-module.jpg"
tags:
    - nuxt
---

## 1. 路由
`nuxt.js`会根据`pages`目录下的文件内容自动生成路由配置
- 路由结构

`pages`目录下的结构如下
```
pages/
--| users/
----| index.vue
----| _id.vue
--| users.vue
--| _slug/
----| index.vue
--| index.vue
```
那么nuxt自动生成的路由配置如下：

```js
router: {
  routes: [
    {
      name: 'index',
      path: '/',
      component: 'pages/index.vue'
    },
    {
      path: '/users',
      component: 'pages/users.vue',
      children: [
        {
          name: 'users',
          path: '',
          component: 'pages/users/index.vue'
        },
        {
          name: 'users-id',
          path: ':id',
          component: 'pages/users/_id.vue'
        }
      ]
    },
    {
      name: 'slug',
      path: '/:slug',
      component: 'pages/_slug/index.vue'
    }
  ]
}
```
**嵌套路由需要增加`<nuxt-child/>`用于显示子视图的内容**
- 路由参数校验
在`pages/users/_id.vue`

```js
export default {
  validate ({ params }) {
    // Must be a number
    return /^\d+$/.test(params.id)
  }
}
```
如果检验的返回值不为true，Nuxt.js将自动加载显示404错误页面

- 过渡动画

Nuxt.js使用vue.js里面的`<transiton></transiton>`实现路由切换的过渡动效。过渡动画分为两种：全局过渡动画和页面过渡动画

全局过渡动画：

在`assets/main.css`里面添加

```css
.page-enter-active, .page-leave-active {
  transition: opacity .5s;
}
.page-enter, .page-leave-active {
  opacity: 0;
}
```
然后在nuxt.config.js添加：

```js
module.exports = {
  css: [
    'assets/main.css'
  ]
}
```

页面过渡动画：

在`assets/main.css`里面添加

```css
.test-enter-active, .test-leave-active {
  transition: opacity .5s;
}
.test-enter, .test-leave-active {
  opacity: 0;
}
```

然后再需要使用的页面组件添加：

```js
export default {
  transition: 'test'
}
```

## 2. 视图

- ## 布局

### 默认布局

Nuxt.js默认布局使用的是`layouts/default.vue`,可以修改该文件来扩展布局。

*`<nuxt />`为显示页面的主体内容*

### 错误页面

*通过编辑`layouts/error.vue`*文件来定制化错误页面

示例：

```html
<template>
  <div class="container">
    <h1 v-if="error.statusCode === 404">Page not found</h1>
    <h1 v-else>An error occurred</h1>
    <nuxt-link to="/">Home page</nuxt-link>
  </div>
</template>

<script>
export default {
  props: ['error'],
  layout: 'blog' // you can set a custom layout for the error page
}
</script>
```

### 自定义布局

目录下的每个第一级文件将创建一个可通过`layout`页面组件中的属性访问的自定义布局。

示例`layouts/blog.vue`：

```html
<template>
  <div>
    <div>My blog navigation bar here</div>
    <nuxt/>
  </div>
</template>
```

然后在`pages/posts.vue`

```html
<script>
  export default {
    layout: 'blog'
  }
</script>
```

### 页面

每个Page组件就是一个Vue组件，但是它有特殊配置项

```html
<template>
  <h1 class="red">Hello {{ name }}!</h1>
</template>

<script>
export default {
  asyncData (context) {
    // called every time before loading the component
    return { name: 'World' }
  },
  fetch () {
    // The fetch method is used to fill the store before rendering the page
  },
  head () {
    // Set Meta Tags for this Page
  },
  // and more functionality to discover
  ...
}
</script>

<style>
.red {
  color: red;
}
</style>
```
特殊配置项如下：

![](https://note.youdao.com/yws/api/personal/file/WEB6e8312c318239a2479a62318d35d5533?method=download&shareKey=eb224c7543ae7ac25776415678fb94db)

## 3. 异步数据

- ### asyncData 方法
`asyncData`方法只有在页面组件中调用，可以在服务端或路由更新之前调用。该方法第一个参数为当前页面的上下文对象，该方法可以用来获取数据，返回值会被融合到组件的`data`方法中

```js
export default {
  asyncData ({ params }) {
    return axios.get(`https://my-api/posts/${params.id}`)
    .then((res) => {
      return { title: res.data.title }
    })
  }
}
```

- ### 错误处理

上下文对象`content`中有一个`error(params)`的方法，可以通过调用该方法显示错误信息页面，`params.statusCode`用于自定服务端返回的请求状态码。

示例：

```js
export default {
  asyncData ({ params, error }) {
    return axios.get(`https://my-api/posts/${params.id}`)
    .then((res) => {
      return { title: res.data.title }
    })
    .catch((e) => {
      error({ statusCode: 404, message: 'Post not found' })
    })
  }
}
```

## 4.插件
- ### 使用第三方模块
在项目发开发中，插件引入比较常见，如果是通过npm装的包，比如
````
npm install axios --save
````
可以直接在组件中使用

```html
<template>
  <h1>{{ title }}</h1>
</template>

<script>
import axios from 'axios'

export default {
  async asyncData ({ params }) {
    let { data } = await axios.get(`https://my-api/posts/${params.id}`)
    return { title: data.title }
  }
}
</script>
```
如果页面多次引入同一个axios，将会多次打包在引入的页面中，可以在`nuxt.config.js`里面配置`build.vendor`来解决：

```js
module.exports = {
  build: {
    vendor: ['axios']
  }
}
```
- ### 使用Vue插件
在`plugins`文件夹新建`vue-notifications.js`

```js
import Vue from 'vue'
import VueNotifications from 'vue-notifications'

Vue.use(VueNotifications)
```

`nuxt.config.js`内配置

```js
module.exports = {
  plugins: ['~/plugins/vue-notifications']
}
```

- ### 只在浏览器中运行
`nuxt.config.js`内配置

```js
module.exports = {
  plugins: [
    { src: '~/plugins/vue-notifications', ssr: false }
  ]
}
```
## 5. vuex状态树
使用vuex有两种方式：

- ### 普通方式
普通方式和vue使用vuex没有多大的区别，在`store`目录下添加`index.js`文件

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = () => new Vuex.Store({

  state: {
    counter: 0
  },
  mutations: {
    increment (state) {
      state.counter++
    }
  }
})

export default store
``` 

- ### 模块方式
模块方式在`store`目录下的`.js`文件会被转换成为指定命名的子模块
示例如下：
`store/index.js`

```js
export const state = () => ({
  counter: 0
})

export const mutations = {
  increment (state) {
    state.counter++
  }
}
```

`store/todos.js`

```js
export const state = () => ({
  list: []
})

export const mutations = {
  add (state, text) {
    state.list.push({
      text: text,
      done: false
    })
  },
  remove (state, { todo }) {
    state.list.splice(state.list.indexOf(todo), 1)
  },
  toggle (state, todo) {
    todo.done = !todo.done
  }
}
```

最终生成的状态输如下：

```js
new Vuex.Store({
  state: { counter: 0 },
  mutations: {
    increment (state) {
      state.counter++
    }
  },
  modules: {
    todos: {
      state: {
        list: []
      },
      mutations: {
        add (state, { text }) {
          state.list.push({
            text,
            done: false
          })
        },
        remove (state, { todo }) {
          state.list.splice(state.list.indexOf(todo), 1)
        },
        toggle (state, { todo }) {
          todo.done = !todo.done
        }
      }
    }
  }
})
```