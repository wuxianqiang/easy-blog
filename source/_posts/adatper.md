---
title: 设计模式之适配器模式
date: 2019-08-26 22:46:43
tags: JavaScript
---
{% asset_img banner.png 图片 %}

适配器模式的作用是解决兼容问题的。

<!-- more -->

在使用Vue的时候通常使用computed计算属性去处理数据然后再显示到页面中。

```js
<body>
  <div id="root">
    <p>{{ name }}</p>
    <p>{{ upperName }}</p>
  </div>

  <script>
    let vm = new Vue({
      el: '#root',
      data: {
        name: 'abcd'
      },
      computed: {
        // upperName适配器模式，把小写字母转换为大写字母
        upperName () {
          return this.name.toUpperCase()
        }
      },
    })
</script>
</body>
```

代码中使用的 upperName 就是适配器，把原始数据适配成我们想要的数据，这里的功能是把字母转换成大写。



电脑的适配器大家都知道，转换电压的，然后让其他设备可以正常使用，比如我有个一个适配器把220V电压转换成12V电压。

```js
// 原始电压220V
class Power {
  charge () {
    return '220w'
  }
}

// Adaptor适配器，把220V转换成12V
class Adaptor {
  constructor () {
    this.power = new Power()
  }
  charge () {
    let v = this.power.charge;
    return `${v()}=>12v`
  }
}

// 客户端要使用12V
class Client {
  constructor () {
    this.adaptor = new Adaptor()
  }
  use() {
    console.log(this.adaptor.charge())
  }
}

let client = new Client();
client.use()
```
这就是适配器模式的使用，也许很多地方你已经用到了。
