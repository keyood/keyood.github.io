---
layout:     post
title:      "liveServer"
subtitle:   "简单好用的静态服务器"
date:       2019-05-28
author:     "keyood"
header-img: "img/post-bg-vscode.png"
tags:
    - vscode
---

> 很多时候碰到打包代码后没法验证正确性的情况，往往要麻烦运维的同学发布到测试站，大大的影响了效率。我起初的做法是本地安装一个`ngnix`。但是，不同的的项目配置不同，每次都要修改配置文件，这个过程极度浪费时间，这时候就想到`VSCode`有没有什么插件解决这个问题，一番搜索，还真的有^_^

### VSCode 使用方式

#### 启动方式

方法一 ：安装 liveServer 插件后，编辑器下方会出现快速开始的按钮，点击后，会自动打开默认浏览器，端口号默认5500

  ![quick-start](https://github.com/ritwickdey/vscode-live-server/raw/master/images/Screenshot/vscode-live-server-statusbar-3.jpg "快速开始")
  
方法二 ： 在文件导航中选中静态文件html，或当前编辑的文件中，右键呼出的快捷功能

方法三： 快捷键
   - Windows 打开 (alt+L, alt+O)，关闭 (alt+L, alt+C)
   - MAC 打开 (cmd+L, cmd+O)，关闭 (cmd+L, cmd+C)
  
方法四 ：F1 or ctrl+shift+P 打开命令面板，选择打开或关闭Live Server

#### 配置

  既然是插件，当然是可配置的, 选择想要修改的设置

  ```js
  {
    "liveServer.settings.AdvanceCustomBrowserCmdLine": null, // 快捷键
    "liveServer.settings.ChromeDebuggingAttachment": false, // debug
    "liveServer.settings.CustomBrowser": "chrome", // 默认浏览器
    "liveServer.settings.NoBrowser": false, // 不打开浏览器
    "liveServer.settings.donotShowInfoMsg": true, // 提不示信息
    "liveServer.settings.donotVerifyTags": false, // 当body、head等标签缺失时，不提示
    "liveServer.settings.fullReload": false, // 修改css后会reload 
    "liveServer.settings.ignoreFiles": [ // 忽视的文件
      ".vscode/**",
      "**/*.scss",
      "**/*.sass",
      "**/*.ts"
    ],
    "liveServer.settings.mount": [], // 将文件夹加入路由 [['/components', './node_modules']]
    "liveServer.settings.multiRootWorkspaceName": null, // vscode 打开多项目时，指定目标项目名
    "liveServer.settings.https": {  // https
      "enable": false,
      "cert": "",
      "key": "",
      "passphrase": ""
    },
    "liveServer.settings.root": "/dist/", // 目录
    "liveServer.settings.file": "index.html", // 入口文件
    "liveServer.settings.host": "127.0.0.1", // host
    "liveServer.settings.port": "5500", // 端口
    "liveServer.settings.proxy": { // 反向代理
      "enable": true, // 开启代理.
      "baseUri": "/api/", // baseUrl, 匹配到 /api/ 都会被代理
      "proxyUri": "http://xxx/" // 代理的目标url
    }
  }
  ```

### NODE 使用方式

当然了，对于没有使用 vsCode 的同学页也可以使用 npm 安装到项目

```bash
npm i -D live-server
```

根目录下新建 `liveServer.js`

```js
var liveServer = require("live-server");

var params = {
    port: 8181,
    host: "0.0.0.0",
    root: "/public",
    open: false,
    ignore: 'scss,my/templates',
    file: "index.html",
    wait: 1000,
    mount: [['/components', './node_modules']],
    logLevel: 2, // 0 = errors only, 1 = some, 2 = lots
    middleware: [function(req, res, next) { next(); }] // Takes an array of Connect-compatible middleware that are injected into the server middleware stack
};
liveServer.start(params);
```

package.json 中配置启动项

```text
  "scripts": {
    "start": "node ./liveServer.js",
  }
```

以上
