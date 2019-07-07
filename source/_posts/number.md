---
title: 泄密Number的数据类型转换
date: 2019-07-07 14:54:07
tags: JavaScript
---
{% asset_img banner.jpg 图片 %}

先考考你这几个题目

<!-- more -->

```js
Number('前端精髓')
Number(null)
Number(undefined)
Number([])
Number([1])
Number([1, 2])
Number(Number(new Date()))
```

感觉还可以吗？

如果你理解Number函数的转换规则，那么应该很容易的，但是里面的转换规则还是挺多的，一不留神就出错，那就详细分析一下。

如果是把布尔值传递给Number函数，将会得到0和1。

```js
Number(false)
// 0
Number(true)
// 1
```


如果是把null和undefined传递给Number函数，将会得到0和NaN。

```js
Number(null)
// 0
Number(undefined)
// NaN
```

如果是把字符串传递给Number函数，那肯定是NaN。

```
Number('前端精髓')
// NaN
```


如果是对象{}传递给Number函数，那肯定是NaN了。

```js
let obj = {}
obj.toString()
// "[object Object]"
Number("[object Object]")
// NaN
```

对象的转换规则其实是这样的：会调用对象的toString方法得到结果，然后再传入Number方法中。

对于数组的转换规则呢，你要先知道数组调用toString会发生什么情况？

```js
[].toString()
// ''
[1].toString()
// '1'
[1,2].toString()
// '1,2'
```

然后我们再调用Number方法就可以了

```js
Number('')
// 0
Number('1')
// 1
Number('1,2')
// NaN
```
