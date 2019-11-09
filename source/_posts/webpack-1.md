---
title: 实现简单的webpack
date: 2019-11-02 19:46:09
tags: Webpack
---
{% asset_img banner.png 图片 %}
<!-- more -->
创建一个简单的webpack配置文件
```js
const path = require('path')

module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

通过npx webpack命令去打包，打包之后的代码如下：
```js
(function (modules) {
  var installedModules = {};
  function __webpack_require__(moduleId) {
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    var module = installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    };
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    return module.exports;
  }
  // 此处省略部分代码
  return __webpack_require__(__webpack_require__.s = "./src/index.js");
})
  ({
    "./src/a.js":
      (function (module, exports) {
        eval("exports.a = 'a模块'\r\n\n\n//# sourceURL=webpack:///./src/a.js?");
      }),
    "./src/index.js":
      (function (module, exports, __webpack_require__) {
        eval("const a = __webpack_require__(/*! ./a */ \"./src/a.js\")\r\nconsole.log(a)\r\n\n\n//# sourceURL=webpack:///./src/index.js?");
      })
  });
```
在打包的代码里面`__webpack_require__`方法是模块的加载实现，包括了ES6和commonJS的两种模块加载实现。这个模块的实现代码不会改变。会改变的是这个自执行函数的参数。



自执行函数的参数是个对象，属性名称是模块的路径，属性值是经过处理之后的代码。
```js
// 源代码：exports.a = 'a模块'
// 打包后代码如下(function (module, exports, __webpack_require__) {源代码})
{
  "./src/a.js":
    (function (module, exports) {
      eval("exports.a = 'a模块'\r\n\n\n//# sourceURL=webpack:///./src/a.js?");
    }),
  "./src/index.js":
    (function (module, exports, __webpack_require__) {
      eval("const a = __webpack_require__(/*! ./a */ \"./src/a.js\")\r\nconsole.log(a)\r\n\n\n//# sourceURL=webpack:///./src/index.js?");
    })
}
```

源代码的require会被替换成`__webpack_require__`这个函数，还有路径`./a`会被替换成`./src/a.js`形式的路径。相当于运行目录的路径。





要实现webpack这个npm包，我们需要先创建package.json文件，通过bin字段指定运行命令时的可执行文件。
```js
{
  "name": "webpack",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "bin": {
    "webpack": "./bin/webpack.js"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

在可执行文件中有如下代码，会执行一个Compiler编译函数。
```js
#! /usr/bin/env node

const path = require('path')
const configPath = path.resolve('webpack.config.js')
const config = require(configPath)
const compiler = new Compiler(config)
const Compiler = require('../lib/Compiler')
compiler.run()
```

具体的编译过程是先分析入口文件，通过抽象语法树分析入口文件的代码是否包含其他的模块，将依赖的模块都放到一个数组中，然后递归分析源代码，维护了这个自执行函数的参数，最后通过ejs将处理成打包后的代码。
```js
let fs = require('fs')
let path = require('path')
const babylon = require('babylon');
const traverse = require('babel-traverse').default;
const generate = require('babel-generator').default;

class Compiler {
  constructor (config) {
    this.config = config
    this.modules = {}
    this.entry = config.entry
    this.root = process.cwd()
  }
  buildModule(modulePath, isEntry) {
    let source = this.getSource(modulePath)
    let moduleName = './' + path.relative(this.root, modulePath).replace(path.sep, '/');
    let parentName = path.dirname(moduleName)
    if (isEntry) {
      this.entryId = moduleName
    }
    let {r, dependencies} = this.parse(source, parentName)
    this.modules[moduleName] = r
    dependencies.forEach(dep => {
      this.buildModule(path.join(this.root, dep), false)
    })
  }
  parse (source, parentDir) {
    const ast = babylon.parse(source);
    let dependencies = [];
    traverse(ast, {
      CallExpression(p) {
        let node = p.node;
        if (node.callee.name === 'require') {
          node.callee.name = '__webpack_require__';
          let value = node.arguments[0].value;
          let ext = path.extname(value);
          value = ext ? value : `${value}.js`;
          value = path.join(parentDir, value);
          value = './' + value.replace(path.sep, '/');
          node.arguments[0].value = value;
          dependencies.push(value);
        }
      }
    });
    let r = generate(ast);
    return { r: r.code, dependencies }
  }
  getSource(modulePath) {
    let content = fs.readFileSync(modulePath, 'utf8')
    return content
  }
  run () {
    this.buildModule(path.resolve(this.root, this.entry), true)
    this.emitFile()
  }
  emitFile () {
    let ejs = require('ejs');
    let templateStr = this.getSource(path.resolve(__dirname, './template.ejs'));
    let str = ejs.render(templateStr, {
      entryId: this.entryId,
      modules: this.modules
    })
    let { filename, path: p } = this.config.output;
    this.assets = {
      [filename]: str
    }
    Object.keys(this.assets).forEach(key => {
      fs.writeFileSync(path.join(p, key), this.assets[key]);
    })
  }
}

module.exports = Compiler
```
打包的样板文件
```js
(function (modules) {
  var installedModules = {};
  function __webpack_require__(moduleId) {
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    var module = installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    };
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    return module.exports;
  }
  return __webpack_require__(__webpack_require__.s = "<%-entryId%>");
})
  ({
    <% for (let key in modules) { %>
      "<%-key%>": (function(module, exports, __webpack_require__) {
        eval(
          `<%-modules[key]%>`
        )
      }),
    <% } %>
  });
```
