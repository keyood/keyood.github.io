---
layout:     post
title:      "编译器原理学习"
subtitle:   "编译器原理学习及demo"
date:       2019-02-15
author:     "keyood"
header-img: "img/post-bg-js-module.jpg"
tags:
    - compiler
---

1. 解析(parse)：将代码字符串解析成抽象语法树AST
2. 变换(transform) ：对抽象语法树进行变换操作
3. 再建(generate)：根据变换后的抽象语法树再生成代码字符串

 ![compiler](/../img/in-post/20190215/compiler.png)

### 什么是抽象语法树abstract syntax tree - AST

AST 是为了方便计算机理解源代码、用于表达源代码语法结构的树状结构，由称作节点（Node）的数据结构组成。

![compiler](/../img/in-post/20190215/ast.png)

所有的抽象语法树（AST）根节点都是 Program 节点，这个节点包含了所有的最顶层语句。这个例子中，包含了 2 部分：
1.一个变量声明，将标识符 a 赋值为数值 3。
2.一个二元表达式语句，描述为标志符为 a，操作符 + 和数值 5

AST 的生产通常包含两个步骤

- 词法分析

 词法分析，也叫做扫描scanner。它读取我们的代码(字符串的形式)，然后把它们按照预定的规则合并成一个个的标识tokens。同时，它会移除空白符，注释，等。最后，整个代码将被分割进一个tokens列表（或者说一维数组）

![token](/../img/in-post/20190215/token.png)

- 语法分析
 语法分析，也解析器。它会将词法分析出来的数组转化成树形的表达形式。同时，验证语法，语法如果有错的话，抛出语法错误

![parser](/../img/in-post/20190215/parser.png)

- 变换

 对AST进行遍历，依据变换规则改变需要的节点信息，返回新的AST
 基本的树的遍历，分为 DFS 和 BFS。AST 树的遍历使用 DFS，对于每个结点，有进入结点和退出结点两个时刻（对应于递归调用的入栈与出栈），所以我们可以在这两个时机访问结点

![transform](/../img/in-post/20190215/transform.png)

 [示例地址](https://github.com/keyood/compiler-demo)