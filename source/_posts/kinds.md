---
title: 理解JavaScript中的多态
date: 2019-09-14 18:06:23
tags: JavaScript
---

{% asset_img banner.png 图片 %}

<!-- more -->

多态的实际含义是：同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结果。


假设我们要编写一个地图应用，现在有两家可选的地图 API 提供商。

```js
const googleMap = {
  show: function () {
    console.log('谷歌地图');
  }
};
const baiduMap = {
  show: function () {
    console.log('百度地图');
  }
};
const renderMap = function (type) {
  if (type === 'google') {
    googleMap.show();
  } else if (type === 'baidu') {
    baiduMap.show();
  }
};
renderMap('google'); // 输出：谷歌地图
renderMap('baidu'); // 输出：百度地图
```

同一个方法renderMap，根据传入的参数不同，作出了各自不同的反应。调用各家地图的show方法。(同一个接口可以不同实现)



多态背后思想是将“做什么”和“谁去做以及怎样去做”分离开来，也就是将“不变的事物”与 “可能改变的事物”分离开来。



下面是改写后的代码，首先我们把不变的部分隔离出来，地图都会有show方法的API。


```js
const googleMap = {
  show: function () {
    console.log('谷歌地图');
  }
};
const baiduMap = {
  show: function () {
    console.log('百度地图');
  }
};

const renderMap = function (map) {
  if (map.show instanceof Function) {
    map.show();
  }
};
renderMap(googleMap); // 输出：谷歌地图
renderMap(baiduMap); // 输出：百度地图
```

方法 renderMap 根据传入的地图对象进行调用显示。这就是对象的多态性。
