---
title: 详细理解script标签
date: 2019-07-06 22:31:50
tags: JavaScript
---

{% asset_img defer.png 图片 %}

说到script标签，大家耳熟能详的就是src属性了，但是你是否记得它有一些其它属性呢？接下来我仔细盘点，看看是否知道它们的含义。

<!-- more -->

```js
async
charset
defer
src
type
```

常用的就是src属性，通过指向一个外部地址来加载脚本，如果用了src属性，那么标签中的内容会被忽略，不会被执行的！！！

```js
<script src="test.js">
  console.log('不会执行')
</script>
```

script标签是没有跨域问题，可以加载任何网站的脚本，和img标签非常类似，所以通常也会用来解决跨域问题，就是人们通常所说的JSONP。

charset表示字符集，是可选的参数，默认utf-8编码。type是内容类型，是可选的参数，默认是text/javascript。 这两个属性了解即可，略过......

默认script标签都是同步加载脚本的，因为脚本之间存在依赖关系，只要前面的执行完毕才能执行后面的代码。但是如果脚本之间没有必然的关系，那么一般会选择异步加载脚本来提高效率，需要使用async属性。

```js
<script src="test.js" async>
</script>
```

async要和src属性配合使用。因为只对外部脚本起作用，如果你是在script标签里面书写脚本，该脚本是不会异步加载！！！

还有一个是defer属性，表示延迟执行，具体意思是当页面解析和显示之后再执行。这个属性也是针对外部属性才起作用。

异步脚本一定会在页面的load事件前执行完毕，但是可能会在DOMContentLoaded事件触发之前或者之后执行。
