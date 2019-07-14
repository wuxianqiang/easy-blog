---
title: 看看webpack打包优化
date: 2019-07-14 22:32:39
tags: Webpack
---

{% asset_img banner.png 图片 %}

webpack 打包策略，主要有下列几种方法。

<!-- more -->

2. CDN
3. DllPlugin
4. splitChunks
5. happypack

下面详细介绍使用的流程。

## CDN

通常使用的第三方模块很大时，但是又想提高打包速度，可以通过CDN引入第三方包来处理。

```js
const AddAssetHtmlCdnWebpackPlugin = require('add-asset-html-cdn-webpack-plugin');
module.exports = {
  externals: {
    'query': '$'
  },
  plugins: [
    new AddAssetHtmlCdnWebpackPlugin(true, {
      'jquery': 'http://code.jquery.com/jquery-3.4.1.js'
    })
  ]
};
```

这里的webpack配置，参数是`external`表示代码中`import $ from jquery`使用到jquery包将不会打包。而 CDN 是通过 webpack 插件导入到HTML中的。

## DllPlugin

我们称为动态链接库，通过生成配置文件和模块进行建立关联。

首先我们要生成配置文件，比如打包react和react-dom需要很长时间，我们可以通过把这两个包提前打包好，使用的时候直接引入这个已经打包好的包就可以了。

新建一个配置文件`webpack.dll.js`。
```
const webpack = require('webpack')
const path = require('path')

module.exports = {
  entry: {
    react: ['react', 'react-dom']
  },
  output: {
    library:  'react',
    filename: '[name].dll.js'
  },
  plugins: [
    new webpack.DllPlugin({
      name: 'react',
      path: path.resolve(__dirname, 'dist/manifest.json')
    })
  ]
}
```
package.json 增加一个脚本
```
 "dll": "webpack --config webpack.dll.js --mode=development"
```
运行`npm run dll`，然后打包出的文件 react.dll.js 和 manifest.json

然后还需要修改你的webpack配置文件webpack.config.js
```
const AddAssetHtmlPlugin = require('add-asset-html-webpack-plugin')

plugins: [
    new webpack.DllReferencePlugin({
      manifest: path.resolve(__dirname, 'dist/manifest.json')
    }),
    new AddAssetHtmlPlugin({ filepath: path.resolve(__dirname, 'dist/react.dll.js') })
]
```
其中使用到了一个插件就是在HTML中引入静态资源的插件。

## splitChunks

这个打包优化项目会在多入口打包用到，意思就是在多页面应用通常使用到了相同的模块，我们可以把公共模块提取出来。

只需要在webpack中增加如下代码，无需任何插件配置。
```
module.exports = {
  //...
  optimization: {
    splitChunks: {
      chunks: 'async',
      minSize: 30000,
      maxSize: 0,
      minChunks: 1,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      automaticNameDelimiter: '~',
      automaticNameMaxLength: 30,
      name: true,
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true
        }
      }
    }
  }
};
```
## happypack

happypack 就能让Webpack把任务分解给多个子进程去并发的执行，子进程处理完后再把结果发送给主进程。

```
// @file: webpack.config.js
const HappyPack = require('happypack');

exports.module = {
  rules: [
    {
      test: /.js$/,
      // 1) replace your original list of loaders with "happypack/loader":
      // loaders: [ 'babel-loader?presets[]=es2015' ],
      use: 'happypack/loader',
      include: [ /* ... */ ],
      exclude: [ /* ... */ ]
    }
  ]
};

exports.plugins = [
  // 2) create the plugin:
  new HappyPack({
    // 3) re-add the loaders you replaced above in #1:
    loaders: [ 'babel-loader?presets[]=es2015' ]
  })
];
```

打包策略还有很多，这是比较主流的打包方式，很多项目通常会使用到，当然webpack也自带了一些优化项，比如tree-shaking和scope-hoisting，大家值得参考。
