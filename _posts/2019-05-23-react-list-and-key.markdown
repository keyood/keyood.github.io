---
layout:     post
title:      "列表 & key"
subtitle:   "列表渲染时为什么不建议将索引当作key"
date:       2019-05-23
author:     "keyood"
header-img: "img/post-bg-react.png"
tags:
    - react
---

key 帮助 React 识别哪些元素改变了，比如被添加或删除。因此你应当给数组中的每一个元素赋予一个`确定的`(stable)标识。

> 一个元素的 key 最好是这个元素在列表中拥有的一个`独一无二`的字符串。通常，我们使用来自数据 id 来作为元素的 key

```js
const todoItems = todos.map((todo) =>
  <li key={todo.id}>
    {todo.text}
  </li>
)
```

> 当元素没有确定 id 的时候，万不得已你可以使用元素索引 index 作为 key

```js
const todoItems = todos.map((todo, index) =>
  // Only do this if items have no stable IDs
  <li key={index}>
    {todo.text}
  </li>
)
```
