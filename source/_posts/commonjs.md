---
title: Node对CommonJS模块的实现
date: 2019-07-26 22:21:19
tags: Node
---

{% asset_img banner.png 图片 %}

在Node中，每个文件模块都是一个对象，它的定义如下：

<!-- more -->

```js
function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  if (parent && parent.children) {
      parent.children.push(this);
  }
  this.filename = null;
  this.loaded = false;
  this.children = [];
}
```

编译和执行是引入文件模块的最后一个阶段。定位到具体的文件后，Node会新建一个模块对象，然后根据路径载入并编译。对于不同的文件扩展名，其载入方法也有所不同，具体如下所示。



1. `.js` 文件。通过 fs 模块同步读取文件后编译执行。
2. `.node` 文件。这是用C/C++编写的扩展文件，通过 `dlopen()` 方法加载最后编译生成 的文件。
3. `.json` 文件。通过 fs 模块同步读取文件后，用 `JSON.parse()` 解析返回结果。其余扩展名文件。它们都被当做`.js`文件载入。

每一个编译成功的模块都会将其文件路径作为索引缓存在 `Module._cache` 对象上，以提高二次引入的性能。

根据不同的文件扩展名，Node会调用不同的读取方式，如`.json`文件的调用如下：

```js
Module._extensions['.json'] = function(module, filename) {
  var content = NativeModule.require('fs').readFileSync(filename, 'utf8');
  try {
      module.exports = JSON.parse(stripBOM(content));
  } catch (err) {
      err.message = filename + ': ' + err.message;
      throw err;
  }
};
```

在确定文件的扩展名之后，Node将调用具体的编译方式来将文件执行后返回给调用者。


回到CommonJS模块规范，我们知道每个模块文件中存在着 `require` 、 `exports` 、 module 这3个变量，但是它们在模块文件中并没有定义，那么从何而来呢？

甚至在Node的API文档中，我们知道每个模块中还有 `__filename` 、 `__dirname` 这两个变量的存在，它们又是从何而来的呢？

事实上，在编译的过程中，Node对获取的JavaScript文件内容进行了头尾包装。在头部添加了 `(function (exports, require, module, __filename, __dirname) {\n` ，在尾部添加了 `\n})`; 。一个正常的JavaScript文件会被包装成如下的样子：

```js
(function (exports, require, module, __filename, __dirname) {
  var math = require('math');
  exports.area = function (radius) {
    return Math.PI * radius * radius;
  };
});
```

这样每个模块文件之间都进行了作用域隔离。包装之后的代码会通过 `vm` 原生模块的 `runInThisContext()` 方法执行（类似 `eval` ，只是具有明确上下文，不污染全局），返回一个具体的 `function` 对象。

最后，将当前模块对象的 `exports` 属性、 `require()` 方法、 `module` （模块对象自身），以及在文件定位中得到的完整文件路径和文件目录作为参数传递给这个 `function()` 执行。

这就是这些变量并没有定义在每个模块文件中却存在的原因。在执行之后，模块的 exports 属性被返回给了调用方。 exports 属性上的任何方法和属性都可以被外部调用到，但是模块中的其余 变量或属性则不可直接被调用。

至此， `require` 、 `exports` 、 `module` 的流程已经完整，这就是Node对CommonJS模块规范的实现。

```js
const path = require('path');
const fs = require('fs');
const vm = require('vm');

function Module(id) {
  this.id = id;
  this.exports = {};
  this.loaded = false;
  this.filename = null;
}

Module._cache = Object.create(null);
Module._extensions = Object.create(null);

Module.wrap = function (script) {
  return Module.wrapper[0] + script + Module.wrapper[1];
};

Module.wrapper = [
  '(function (exports, require, module, __filename, __dirname) { ',
  '\n});'
];

Module._load = function (request) {
  var filename = Module._resolveFilename(request);
  var cachedModule = Module._cache[filename];
  if (cachedModule) {
    return cachedModule.exports;
  }
  var module = new Module(filename);
  Module._cache[filename] = module;
  tryModuleLoad(module, filename);
  return module.exports;
};

Module._resolveFilename = function (request) {
  var filename = Module._findPath(request);
  if (!filename) {
    var err = new Error(`Cannot find module '${request}'`);
    err.code = 'MODULE_NOT_FOUND';
    throw err;
  }
  return filename;
};

Module._findPath = function (request) {
  var filename = path.resolve(request);
  return filename;
};

function tryModuleLoad(module, filename) {
  var threw = true;
  try {
    module.load(filename);
    threw = false;
  } finally {
    if (threw) {
      delete Module._cache[filename];
    }
  }
}

Module.prototype.load = function (filename) {
  this.filename = filename;
  var extension = path.extname(filename) || '.js';
  if (!Module._extensions[extension]) extension = '.js';
  Module._extensions[extension](this, filename);
  this.loaded = true;
};

Module._extensions['.js'] = function (module, filename) {
  var content = fs.readFileSync(filename, 'utf8');
  module._compile(content, filename);
};

Module._extensions['.json'] = function (module, filename) {
  var content = fs.readFileSync(filename, 'utf8');
  try {
    module.exports = JSON.parse(content);
  } catch (err) {
    err.message = filename + ': ' + err.message;
    throw err;
  }
};

Module.prototype._compile = function (content, filename) {
  var wrapper = Module.wrap(content);
  var compiledWrapper = vm.runInThisContext(wrapper);
  var dirname = path.dirname(filename);
  var result;
  result = compiledWrapper.call(this.exports, this.exports, require, this, filename, dirname);
  return result;
};

function require2(path) {
  return Module._load(path);
}

let str = require2('./b.json') // 导入JSON数据
```
