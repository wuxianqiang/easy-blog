---
title: 选择文本
date: 2019-11-30 11:36:53
tags: JavaScript
---
{% asset_img banner.png 图片 %}

文本框都支持 `select()` 方法，这个方法用于选择文本框中的所有文本。在调用 `select()` 方法时，大多数浏览器都会将焦点设置到文本框中。这个方法不接受参数，可以在任何时候被调用。

<!-- more -->

```js
  <input type="text" id="username" value="hello world">
  <script>
    // 给元素添加ID默认会变成全局变量
    username.select()
  </script>
```
{% asset_img select1.png 图片 %}




与 `select()` 方法对应的，是一个 select 事件。在选择了文本框中的文本时，就会触发 select 事件。



## 取得选择的文本

虽然通过 select 事件我们可以知道用户什么时候选择了文本，但仍然不知道用户选择了什么文本。



`selectionStart` 和 `selectionEnd` 是HTML5新添加的两个属性，这两个属性中保存的是基于0的数值，表示所选择文本的范围。


```js
  <input type="text" id="username" value="hello world">
  <script>
    username.select()
    username.onselect = function () {
      let start = this.selectionStart
      let end = this.selectionEnd
      console.log(this.value.substring(start, end))
    }
  </script>
```


## 选择部分文本

HTML5增加了`setSelectionRange()` 方法。所有文本框都有这个方法。这个方法接收两个参数：要选择的第一个字符的索引和要选择的最后一个字符之后的字符的索引。


```js
  <input type="text" id="username" value="hello world">
  <script>
    username.focus()
    username.setSelectionRange(0, 5)
  </script>
```

{% asset_img select2.png 图片 %}


要看到选择的文本，必须在调用 `setSelectionRange()` 之前或之后立即将焦点设置到文本框。
