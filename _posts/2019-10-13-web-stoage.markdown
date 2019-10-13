---
layout: post
title: '浏览器储存方案学习与总结'
subtitle: 'web stoage'
date: 2019-10-13
author: 'keyood'
header-img: 'img/post-bg-laptop.jpg'
tags:
  - 浏览器
---

- [一、IndexedDB Database](#%e4%b8%80indexeddb-database)
  - [基本概念](#%e5%9f%ba%e6%9c%ac%e6%a6%82%e5%bf%b5)
- [二、Cache API](#%e4%ba%8ccache-api)
  - [特性](#%e7%89%b9%e6%80%a7)
  - [方法](#%e6%96%b9%e6%b3%95)
- [三、Service Worker](#%e4%b8%89service-worker)
- [四、Web Storage API](#%e5%9b%9bweb-storage-api)
- [History state](#history-state)
  - [方法](#%e6%96%b9%e6%b3%95-1)
- [五、Application caches](#%e4%ba%94application-caches)
- [六、Notification data](#%e5%85%adnotification-data)
  - [构造方法](#%e6%9e%84%e9%80%a0%e6%96%b9%e6%b3%95)
  - [静态属性](#%e9%9d%99%e6%80%81%e5%b1%9e%e6%80%a7)
  - [实例属性](#%e5%ae%9e%e4%be%8b%e5%b1%9e%e6%80%a7)
  - [方法](#%e6%96%b9%e6%b3%95-2)
  - [事件](#%e4%ba%8b%e4%bb%b6)

## 一、IndexedDB Database

> IndexedDB 是一个事务型数据库系统,基于 JavaScript 的面向对象的数据库，IndexedDB 执行的操作是异步执行的,以免阻塞应用程序

### 基本概念

- 使用 key-value 键值对储存数据
  values 数据可以是结构非常复杂的对象，key 可以是二进制对象
- 事务模式
  任何操作都发生在事务(transaction)中， IndexedDB API 提供了索引(indexes)、表(tables)、指针(cursors)等等，但是所有这些必须是依赖于某种事务的。
- API 异步
  IndexedDB 的 API 不通过 return 语句返回数据，而是需要你提供一个回调函数来接受数据。
- IndexedDB 在结果准备好之后通过 DOM 事件通知用户
  DOM 事件总是有一个类型（type）属性（在 IndexedDB 中，该属性通常设置为 success 或 error）。DOM 事件还有一个目标（target）属性，用来告诉事件是被谁触发的
- 面向对象
  不用二维表来表示集合的关系型数据库
- 不使用结构化查询语言（SQL）
- 遵循同源（same-origin）策略
  IndexedDB 鼓励使用的基本模式如下所示：

打开数据库。
在数据库中创建一个对象仓库（object store）。
启动一个事务，并发送一个请求来执行一些数据库操作，像增加或提取数据等。
通过监听正确类型的 DOM 事件以等待操作完成。
在操作结果上进行一些操作（可以在 request 对象中找到）

## 二、Cache API

> Cache 接口为缓存的 Request / Response 对象对提供存储机制, 定义在 service worker 的标准中, 但是它不必一定要配合 service worker 使用.

### 特性

- 一个域可以有多个命名 Cache 对象
- 不主动更新
- 除非删除，不会过期

### 方法

- match(request, options) 返回一个 Promise 对象，resolve 的结果是跟 Cache 对象匹配的第一个已经缓存的请求
- matchAll(request, options) 返回一个 Promise 对象，resolve 的结果是跟 Cache 对象匹配的所有请求组成的数组
- add(request) 抓取这个 URL, 检索并把返回的 response 对象添加到给定的 Cache 对象
- addAll(requests) 抓取一个 URL 数组，检索并把返回的 response 对象添加到给定的 Cache 对象
- put(request, response) 同时抓取一个请求及其响应，并将其添加到给定的 cache
- delete(request, options) 搜索 key 值为 request 的 Cache 条目。如果找到，则删除该 Cache 条目，并且返回一个 resolve 为 true 的 Promise 对象；如果未找到，则返回一个 resolve 为 false 的 Promise 对象。
- keys(request, options) 返回一个 Promise 对象，resolve 的结果是 Cache 对象 key 值组成的数组

具体使用方式在下方的 Service Worker 中有示例

## 三、Service Worker

> Service workers 本质上充当 Web 应用程序与浏览器之间的代理服务器，也可以在网络可用时作为浏览器和网络间的代理。它们旨在（除其他之外）使得能够创建有效的离线体验，拦截网络请求并基于网络是否可用以及更新的资源是否驻留在服务器上来采取适当的动作。他们还允许访问推送通知和后台同步 API。

下面是一个官方实例，实现缓存图片请求，如下所示为项目结构

![项目结构](/../img/in-post/20191013/sw.png)

html 部分

```html
<body>
  <h1>Lego Star Wars gallery</h1>
  <!-- 插入图片的块 -->
  <section></section>
  <!-- 图片信息 -->
  <script src="image-list.js"></script>
  <!-- 入口文件 -->
  <script src="app.js"></script>
</body>
```

`image-list.js` 定义一组图片信息变量

```js
var Path = 'gallery/'

var Gallery = {
  images: [
    {
      name: 'Darth Vader',
      alt: 'A Black Clad warrior lego toy',
      url: 'gallery/myLittleVader.jpg',
      credit:
        '<a href="https://www.flickr.com/photos/legofenris/">legOfenris</a>, published under a <a href="https://creativecommons.org/licenses/by-nc-nd/2.0/">Attribution-NonCommercial-NoDerivs 2.0 Generic</a> license.'
    },
    {
      name: 'Snow Troopers',
      alt: 'Two lego solders in white outfits walking across an icy plain',
      url: 'gallery/snowTroopers.jpg',
      credit:
        '<a href="https://www.flickr.com/photos/legofenris/">legOfenris</a>, published under a <a href="https://creativecommons.org/licenses/by-nc-nd/2.0/">Attribution-NonCommercial-NoDerivs 2.0 Generic</a> license.'
    },
    {
      name: 'Bounty Hunters',
      alt: 'A group of bounty hunters meeting, aliens and humans in costumes.',
      url: 'gallery/bountyHunters.jpg',
      credit:
        '<a href="https://www.flickr.com/photos/legofenris/">legOfenris</a>, published under a <a href="https://creativecommons.org/licenses/by-nc-nd/2.0/">Attribution-NonCommercial-NoDerivs 2.0 Generic</a> license.'
    }
  ]
}
```

`app.js` 是 js 主要的入口文件，在这里调用 `navigator.serviceWorker .register` 注册 service worker，页面加载后会依次调用 XMLHttpRequest 请求图片，并将请求到图片渲染到页面中。

```js
// 注册 service worker
if ('serviceWorker' in navigator) {
  navigator.serviceWorker
    .register('/sw-test/sw.js', {
      scope: '/sw-test/' // 指定作用域限制在/sw-test/下
    })
    .then(function(reg) {
      if (reg.installing) {
        // 注册中
        console.log('Service worker installing')
      } else if (reg.waiting) {
        // 等待注册
        console.log('Service worker installed')
      } else if (reg.active) {
        // 注册成功
        console.log('Service worker active')
      }
    })
    .catch(function(error) {
      // 注册失败
      console.log('Registration failed with ' + error)
    })
}

// XHR 加载图片

function imgLoad(imgJSON) {
  // 返回Promise
  return new Promise(function(resolve, reject) {
    var request = new XMLHttpRequest()
    request.open('GET', imgJSON.url)
    request.responseType = 'blob'

    request.onload = function() {
      if (request.status === 200) {
        var arrayResponse = []
        arrayResponse[0] = request.response
        arrayResponse[1] = imgJSON
        resolve(arrayResponse)
      } else {
        reject(
          Error(
            "Image didn't load successfully; error code:" + request.statusText
          )
        )
      }
    }

    request.onerror = function() {
      reject(Error('There was a network error.'))
    }

    // Send the request
    request.send()
  })
}

var imgSection = document.querySelector('section')

window.onload = function() {
  // load each set of image, alt text, name and caption
  for (var i = 0; i <= Gallery.images.length - 1; i++) {
    imgLoad(Gallery.images[i]).then(
      function(arrayResponse) {
        var myImage = document.createElement('img')
        var myFigure = document.createElement('figure')
        var myCaption = document.createElement('caption')
        var imageURL = window.URL.createObjectURL(arrayResponse[0])

        myImage.src = imageURL
        myImage.setAttribute('alt', arrayResponse[1].alt)
        myCaption.innerHTML =
          '<strong>' +
          arrayResponse[1].name +
          '</strong>: Taken by ' +
          arrayResponse[1].credit

        imgSection.appendChild(myFigure)
        myFigure.appendChild(myImage)
        myFigure.appendChild(myCaption)
      },
      function(Error) {
        console.log(Error)
      }
    )
  }
}
```

`sw.js` 处理

```js
// service worker 注册之后，浏览器会尝试为你的页面或站点安装并激活它
// install 事件在安装完成之后触发
self.addEventListener('install', function(event) {
  // waitUntil 直到里面的代码执行完毕才完成install
  event.waitUntil(
    // 调用 Cache API 存储资源，Cache API 并不被每个浏览器支持，需要添加polyfill或使用indexDB，它们都是异步的
    caches.open('v1').then(function(cache) {
      return cache.addAll([
        '/sw-test/',
        '/sw-test/index.html',
        '/sw-test/style.css',
        '/sw-test/app.js',
        '/sw-test/image-list.js',
        '/sw-test/star-wars-logo.jpg',
        '/sw-test/gallery/bountyHunters.jpg',
        '/sw-test/gallery/myLittleVader.jpg',
        '/sw-test/gallery/snowTroopers.jpg'
      ])
    })
  )
})
// 每次任何被 service worker 控制的资源被请求到时，都会触发 fetch 事件
self.addEventListener('fetch', function(event) {
  // respondWith() 方法劫持我们的 HTTP 响应
  event.respondWith(
    caches
      .match(event.request) // 匹配请求的资源与缓存的是否相同，这个匹配通过 url 和 vary header进行
      .then(function(response) {
        // caches.match() 总是会返回 resolves
        // 但只有成功的时候有值
        if (response !== undefined) {
          // 如果有匹配，直接返回缓存资源
          return response
        } else {
          // 如果无匹配，请求
          return fetch(event.request)
            .then(function(response) {
              // 拷贝一份响应加入缓存
              let responseClone = response.clone()
              caches.open('v1').then(function(cache) {
                cache.put(event.request, responseClone)
              })
              // 返回原始响应
              return response
            })
            .catch(function() {
              // 请求失败的处理
              return caches.match('/sw-test/gallery/myLittleVader.jpg')
            })
        }
      })
  )
})
```

## 四、Web Storage API

`Storage` 为每一个给定的源（given origin）维持一个独立的存储区域，它是所有 Storage 实例（sessionStorage 和 localStorage）的构造函数。

- sessionStorage 页面会话期间可用
- localStorage 数据会一直在，除非主动清除

下面以 sessionStorage 为例，查看`sessionStorage._proto_`可以看到 Storage 的属性和方法。
![sessionStorage._proto_](/../img/in-post/20191013/storage_proto.png)

```js
// 获取sessionStorage对象的键值对长度
sessionStorage.length

// 按键值对的方式保存数据
sessionStorage.setItem('key', 'value')

// 根据键名获取数据
let data = sessionStorage.getItem('key')

// 接受一个数值 n 作为参数，返回存储对象第 n 个数据项的键名
sessionStorage.key(0) // 'key'

// 根据键名删除保存的数据
sessionStorage.removeItem('key')

// 删除所有保存的数据
sessionStorage.clear()
```

给`Storage`的原型对象添加 removeKey 方法，可通过 localStorage.removeKey 和 sessionStorage.removeKey 访问到。

```js
Storage.prototype.removeKey = function(key) {
  this.removeItem(this.key(key))
}
// 存入数据
sessionStorage.setItem('name', 'jack')
sessionStorage.setItem('age', '20')
sessionStorage.setItem('height', '178')
sessionStorage.setItem('weight', '60')
```

在控制台打印 sessionStorage 对象,得到
![session_object](/../img/in-post/20191013/session_object.png)

```js
console.log(sessionStorage.age) // 20 可以通过键名直接访问
sessionStorage.age = 30 // 30 可以通过键名修改

Object.keys(sessionStorage) // ["height", "name", "age", "weight"]
sessionStorage.removeKey(0) // 将会删除 height
```

![session](/../img/in-post/20191013/session.png)

`StorageEvent`继承 Event 对象，事件在同一个域下的不同页面之间触发，即在 A 页面注册了 storge 的监听处理，只有在跟 A 同域名下的 B 页面操作 storage 对象，A 页面才会被触发 storage 事件, 只有 localStorage 会生效。

```js
window.addEventListener(
  'storage',
  function(e) {
    console.log('storage change', e)
  },
  false
)
```

## History state

> window 对象通过 history 对象提供了对浏览器的会话历史的访问

### 方法

- 向后跳转 `window.history.back()`
- 向前跳转 `window.history.forward()`
- 跳转到 history 中指定的一个点
  向后移动一个页面 `window.history.go(-1)`
  向前移动一个页面 `window.history.go(1)`
- 查看长度属性的值来确定的历史堆栈中页面的数量 `window.history.length`
- 添加历史记录条目(html5) `history.pushState(state, title, url)`

  - 状态对象 — 一个 JavaScript 对象，其序列化后有 640k 的大小限制。

  - 标题 — Firefox 目前忽略这个参数，但未来可能会用到。在此处传一个空字符串应该可以安全的防范未来这个方法的更改。

  - URL — 新的历史 URL 记录。注意，调用 pushState() 后浏览器并不会立即加载这个 URL，但可能会在稍后某些情况下加载这个 URL，比如在用户重新打开浏览器时。新 URL 不必须为绝对路径。如果新 URL 是相对路径，新 URL 必须与当前 URL`同源`，否则 pushState() 会抛出一个异常。该参数是可选的。

- 修改历史记录条目(html5)
  `history.replaceState(state, title, url)`
  替换当前的记录
- window.onpopstate
  window.onpopstate 是 popstate 事件在 window 对象上的事件处理程序，前进、回退时都会触发，state 状态会回退上一个页面栈的状态。

```js
window.onpopstate = function(event) {
  alert(
    'location: ' + document.location + ', state: ' + JSON.stringify(event.state)
  )
}
// 绑定事件处理函数.
history.pushState({ page: 1 }, 'title 1', '?page=1') //添加并激活一个历史记录条目 http://example.com/example.html?page=1,条目索引为1
history.pushState({ page: 2 }, 'title 2', '?page=2') //添加并激活一个历史记录条目 http://example.com/example.html?page=2,条目索引为2
history.replaceState({ page: 3 }, 'title 3', '?page=3') //修改当前激活的历史记录条目 http://ex..?page=2 变为 http://ex..?page=3,条目索引为3
history.back() // 弹出 "location: http://example.com/example.html?page=1, state: {"page":1}"
history.back() // 弹出 "location: http://example.com/example.html, state: null
history.go(2) // 弹出 "location: http://example.com/example.html?page=3, state: {"page":3}
```

## 五、Application caches

> 该特性已经从 Web 标准中删除，虽然一些浏览器目前仍然支持它，但也许会在未来的某个时间停止支持，请尽量不要使用该特性。

## 六、Notification data

> 通知接口用于向用户配置和显示桌面通知。

### 构造方法

`let notification = new Notification(title, options)`

- title 一定会被显示的通知标题
- options(可选) 一个被允许用来设置通知的对象
  - dir : 文字的方向；它的值可以是 auto（自动）, ltr（从左到右）, or rtl（从右到左）
  - lang: 指定通知中所使用的语言。这个字符串必须在 BCP 47 language tag 文档中是有效的。
  - body: 通知中额外显示的字符串
  - tag: 赋予通知一个 ID，以便在必要的时候对通知进行刷新、替换或移除。
  - icon: 一个图片的 URL，将被用于显示通知的图标。

### 静态属性

- permission(只读)
  - granted: 用户已经明确的授予了显示通知的权限。
  - denied: 用户已经明确的拒绝了显示通知的权限。
  - default: 用户还未被询问是否授权; 这种情况下权限将视为 denied。如果触发通知，浏览器会进行弹窗询问，这时点击允许将设置为 granted。

### 实例属性

这些属性仅在 Notification 的实例中有效

- title(只读)
- dir(只读)
- lang(只读)
- body(只读)
- tag(只读)
- icon(只读)

### 方法

- close() 有些浏览器会自动关闭弹出的 Notification，但有些不是
- requestPermission() 请求用户当前来源的权限以显示通知

  ```js
    // 最新的规范基于promise的语法
    Notification.requestPermission().then(function(permission) { ... })
    // callback方式已经弃用
    Notification.requestPermission(callback);
  ```

### 事件

- onclick 每当用户点击通知时被触发。
- onshow 当通知显示的时候被触发。
- onerror 每当通知遇到错误时被触发。
- onclose 当用户关闭通知时被触发。

```html
<button onclick="notifyMe()">Notify me!</button>
```

```js
function notifyMe() {
  var notification

  // 先检查浏览器是否支持
  if (!('Notification' in window)) {
    alert('This browser does not support desktop notification')
  }
  // 检查用户是否同意接受通知
  else if (Notification.permission === 'granted') {
    // If it's okay let's create a notification
    notification = new Notification('Hi there!', {
      body: 'Nice to meet you!'
    })
    console.log(notification.title) // Hi there!
  }

  // 否则我们需要向用户获取权限
  else if (Notification.permission !== 'denied') {
    Notification.requestPermission().then(function(permission) {
      // 如果用户同意，就可以向他们发送通知
      if (permission === 'granted') {
        notification = new Notification('Hi there!', {
          body: 'Nice to meet you!'
        })
      }
    })
  }
  notification.onclick = function(event) {
    console.log('notification onclick', event)
  }
  notification.onshow = function(event) {
    console.log('notification onshow', event)
  }
  notification.onerror = function(event) {
    console.log('notification onerror', event)
  }
  notification.onclose = function(event) {
    console.log('notification onclose', event)
  }
  // 5s 后自动关闭
  setTimeout(function() {
    notification.close()
  }, 5000)
}
```
