---
title: 富文本编辑
date: 2019-11-15 07:45:32
tags: JavaScript
---

{% asset_img banner.png 图片 %}

富文本编辑，又称为WYSIWYG（What You See Is What You Get，所见即所得）。在网页中编辑富文本内容。

<!-- more -->

```js
  <div id="app">hello world</div>
  <script>
    document.designMode = 'on'
  </script>
```


设置 designMode 属性，这个空白的HTML页面可以被编辑，而编辑对象则是该页面 <body> 元素的HTML代码。designMode 属性有两个可能的值：“off” （默认值）和 “on” 。在设置为 “on” 时，整个文档都会变得可以编辑。



除了使整个页面变成编辑状态，还有可以使某个元素变成可编辑的状态，另一种方式就是使用名为 contenteditable 的特殊属性。可以把这个属性应用给页面中的任何元素，然后用户立即就可以编辑该元素。


```js
  <div contenteditable>
    hello world
  </div>
```


这样，元素中包含的任何文本内容就都可以编辑了，就好像这个元素变成了`<textarea>` 元素一样。通过在这个元素上设置 contenteditable 属性，也能打开或关闭编辑模式。



除了通过设置属性的方式，还可以通过JS设置，只需要将元素的contentEditable属性设置成true即可。


```html
  <div id="richedit">
    hello world
  </div>
  <script>
    let div = document.getElementById('richedit');
    div.contentEditable = true;
  </script>
```


contenteditable 属性有三个可能的值：“true” 表示打开、 “false” 表示关闭， “inherit” 表示从父元素那里继承（因为可以在 contenteditable 元素中创建或删除元素）。
