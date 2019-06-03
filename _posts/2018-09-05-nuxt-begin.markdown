---
layout:     post
title:      "nuxt 入门"
subtitle:   "基础知识"
date:       2018-09-05
author:     "keyood"
header-img: "img/post-bg-js-module.jpg"
tags:
    - nuxt
---

## 1. 服务端渲染

 nuxt是一个服务端渲染的框架，在了解它之前，有必要去了解下什么是服务端渲染，什么时候要用到服务端渲染。

服务端渲染也被称为`ssr(Server Side Render)`，不同于客户端渲染，服务端渲染会将完整页面的DOM节结构转成String吐出来，然后到客户端（浏览器）解析渲染。

那么服务端渲染的优势有哪些？

### SEO

现在单页应用使用率比较高，体验好比如目前React，Vue，Angular等前端框架，但是单页应用服务端首次发送到前端只是一个页面的框架，里面是没有具体内容的，里面的内容都是客户端通过js去生成，导致搜索引擎爬不到页面关键词等信息。以下是使用SSR和未使用ssr在浏览器检查网页源代码的表现。

没有ssr：
![没有ssr](https://note.youdao.com/yws/api/personal/file/WEB6f8121a91dd5c7364d075ad1dac2649b?method=download&shareKey=00d480b3dba96251097542a45f33500e)

使用ssr：
![使用ssr](https://note.youdao.com/yws/api/personal/file/WEB820cf64b97a78dfb3b88d67bd3f1cb6d?method=download&shareKey=4834d1d897b6788d2731a1b97f6f9a43)

### 首屏直出

首屏直出指的是在用户首次访问页面不需要等待很长时间，首屏就可以看到页面所有内容。

## 2. 什么是nuxt
以下是引用官方的语言：

```  
Nuxt.js 是一个基于 Vue.js 的通用应用框架。

通过对客户端/服务端基础架构的抽象组织，Nuxt.js 主要关注的是应用的 UI渲染。
```
nuxt集成了vue、vue-router、vuex（当配置了vuex状态树才会引入）、vue-meta

流程图：
![](https://note.youdao.com/yws/api/personal/file/WEB76e353d118550b42c713fc545a2bed58?method=download&shareKey=99720a65003b14656519b40aef3703b6)

## 3. 项目初始化

方法一：使用vue-cli初始化项目(如果未安装，请先使用`npm install -g vue-cli`)
```
$ vue init nuxt-community/starter-template <project-name>
```
其他项目初始化模板

| 模板初始化命令   | 说明    |
| --------        | -----    |
| `nuxt/starter`        | 基本的Nuxt.js项目模板      |
| `nuxt/express`        | Nuxt.js+Express     |
| `nuxt/koa`        | Nuxt.js+Koa2      |
| `nuxt/adonuxt`        | Nuxt.js+adonuxt      |
| `nuxt/micro`        | Nuxt.js+micro      |
| `nuxt/nuxtent`        | Nuxt.js+nuxtent      |

方法二：使用nuxt提供的cli工具

```
// npm 
$ npx nuxt-create-app <project-name>

// yarn
$ yarn create create-app <project-name>
```

接着进入该项目，安装相应的依赖包
```
npm install
```
最后启动项目
```
npm run dev
```
## 4. 项目的目录结构
![](https://note.youdao.com/yws/api/personal/file/WEB33cfb6f09a7033485ea2f70912b15916?method=download&shareKey=0cb7653f3274abfc94010f1591c3bf5f)

项目的目录结构和vue的项目有些类似

- assests
> 资源目录，存放css，图片，js
- components
> 存放组件的地方
- layouts
> 布局目录，应用于布局，默认所有页面都是使用该目录下的`default.vue`模板，错误页面使用`error.vue`模板
- middleware
> 中间件存放的地方
- pages
> 页面存放的地方，该目录下所有`.vue`文件会自动生成对应的路由配置，如：在该目录下有有个`home.vue`,则路由就会相对于应生成`/home`的路由地址
- plugins
> 插件存放的地方，比如配置axios，引入mint-ui都可以在这里配置
- static
> 静态文件存放的地方，webpack不会进行处理，该目录下文件会映射至根路径`/`下
- store
> vuex状态存放的地方，默认不使用vuex，如果在该目录建index.js文件则将使用vuex
- nuxt.config.js
> 项目的个性化配置，用于覆盖默认的配置
## 5. 命令部署

- 命名列表

| 命令   | 描述   |
| --------        | -----    |
| `nuxt`        | 启动一个热加载的web服务器（开发模式）      |
| `nuxt build`        | 利用webpack构建应用程序，压缩js和css（生产模式）     |
| `nuxt start`        | 生产模式启动服务器（运行nuxt build之后）     |
| `nuxt generate` | 生产模式启动服务器（运行nuxt build之后）|

通常开发加入在`package.json`中

```json
"scripts": {
    "dev": "nuxt",
    "build": "nuxt build",
    "start": "nuxt start",
    "generate": "nuxt generate"
  }
```

- 发布部署

发布部署有三种部署应用的方式：服务端渲染部署、静态应用部署、spa部署。

服务端渲染部署操作：
```
npm run build
npm start
```
静态应用部署：
```
npm run generate
```
它将创建一个dist文件夹，其中所有内容可以部署在静态托管站点

单页应用部署（SPA）：
```
npm run build -- --spa
```
或者在`nuxt.config.js`中添加选项`mode: 'spa'`,然后执行
```
npm run build
```
将创建dist文件下的内容部署到服务器就行
