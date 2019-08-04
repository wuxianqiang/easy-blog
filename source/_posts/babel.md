---
title: 快速了解babel工作原理
date: 2019-08-04 08:41:55
tags: JavaScript
---

{% asset_img banner.png 图片 %}

<!-- more -->

## 介绍

Babel 是一个通用的多功能的 JavaScript 编译器。此外它还拥有众多模块可用于不同形式的静态分析。

静态分析是在不需要执行代码的前提下对代码进行分析的处理过程 （执行代码的同时进行代码分析即是动态分析）。静态分析的目的是多种多样的， 它可用于语法检查，编译，代码高亮，代码转换，优化，压缩等等场景。

## 基础

Babel 是 JavaScript 编译器，更确切地说是源码到源码的编译器，通常也叫做“转换编译器（transpiler）”。意思是说你为 Babel 提供一些 JavaScript 代码，Babel 更改这些代码，然后返回给你新生成的代码。

## 抽象语法树（ASTs）

这个处理过程中的每一步都涉及到创建或是操作抽象语法树，亦称 AST。

```js
function square(n) {
  return n * n;
}
```
> AST Explorer[1] 可以让你对 AST 节点有一个更好的感性认识。


这个程序可以被表示成如下的一棵树：

```js
- FunctionDeclaration:
  - id:
    - Identifier:
      - name: square
  - params [1]
    - Identifier
      - name: n
  - body:
    - BlockStatement
      - body [1]
        - ReturnStatement
          - argument
            - BinaryExpression
              - operator: *
              - left
                - Identifier
                  - name: n
              - right
                - Identifier
                  - name: n
```
或是如下所示的 JavaScript Object（对象）：

```js
{
  type: "FunctionDeclaration",
  id: {
    type: "Identifier",
    name: "square"
  },
  params: [{
    type: "Identifier",
    name: "n"
  }],
  body: {
    type: "BlockStatement",
    body: [{
      type: "ReturnStatement",
      argument: {
        type: "BinaryExpression",
        operator: "*",
        left: {
          type: "Identifier",
          name: "n"
        },
        right: {
          type: "Identifier",
          name: "n"
        }
      }
    }]
  }
}
```

你会留意到 AST 的每一层都拥有相同的结构：

```js
{
  type: "FunctionDeclaration",
  id: {...},
  params: [...],
  body: {...}
}
{
  type: "Identifier",
  name: ...
}
{
  type: "BinaryExpression",
  operator: ...,
  left: {...},
  right: {...}
}
```

注意：出于简化的目的移除了某些属性

这样的每一层结构也被叫做 节点（Node）。一个 AST 可以由单一的节点或是成百上千个节点构成。它们组合在一起可以描述用于静态分析的程序语法。


每一个节点都有如下所示的接口（Interface）：

```js
interface Node {
  type: string;
}
```

字符串形式的 type 字段表示节点的类型（如："FunctionDeclaration"，"Identifier"，或 "BinaryExpression"）。每一种类型的节点定义了一些附加属性用来进一步描述该节点类型。

Babel 还为每个节点额外生成了一些属性，用于描述该节点在原始代码中的位置。

```js
{
  type: ...,
  start: 0,
  end: 38,
  loc: {
    start: {
      line: 1,
      column: 0
    },
    end: {
      line: 3,
      column: 1
    }
  },
  ...
}
```

每一个节点都会有 start，end，loc 这几个属性。

## Babel 的处理步骤

Babel 的三个主要处理步骤分别是：解析（parse），转换（transform），生成（generate）。

1. 解析

解析步骤接收代码并输出 AST。这个步骤分为两个阶段：词法分析（Lexical Analysis）和 语法分析（Syntactic Analysis）。

2. 转换

转换步骤接收 AST 并对其进行遍历，在此过程中对节点进行添加、更新及移除等操作。

3. 生成

代码生成步骤把最终（经过一系列转换之后）的 AST 转换成字符串形式的代码，同时还会创建源码映射（source maps）。

代码生成其实很简单：深度优先遍历整个 AST，然后构建可以表示转换后代码的字符串。

用例

我们可以使用npm中的第三方模块来模拟这个过程。尝试修改函数的名字。

```js
// 解析
let esprima = require('esprima')
// 转换
let estraverse = require('estraverse')
// 生成
let escodegen = require('escodegen')

let code = `function code () {}`

// 解析
let ast = esprima.parseScript(code)
// 转换
estraverse.traverse(ast, {
  enter (node) {
    console.log('enter' + node.type)
    if (node.type === 'enterIdentifier') {
      node.name = 'hello'
    }
  },
  leave (node) {
    console.log('enter' + node.type)
  }
})
// 生成
let newCode = escodegen.generate(ast)
References
```

[1] AST Explorer: http://astexplorer.net/
