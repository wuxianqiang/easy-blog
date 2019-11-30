---
title: 接口描述函数类型
date: 2019-11-30 11:59:58
tags: TypeScript
---
{% asset_img banner.png 图片 %}

之前我们通过接口来约束对象，判断对象中的属性数据类型是否符合要求。

<!-- more -->

```js
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}

let squareOptions:SquareConfig = {
  colour: "red",
  width: 100
}
```


接口能够描述JavaScript中对象拥有的各种各样的外形。除了描述带有属性的普通对象外，接口也可以描述函数类型。


```js
let mySearch = function(
  source: string,
  subString: string
):boolean {
  let result = source.search(subString);
  return result > -1;
}
```


为了使用接口表示函数类型，我们需要给接口定义一个调用签名。它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。


```js
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```


对于函数类型的类型检查来说，函数的参数名不需要与接口里定义的名字相匹配。 


```js
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(
  src: string,
  sub: string
): boolean {
  let result = src.search(sub);
  return result > -1;
}
```


函数的参数会逐个进行检查，要求对应位置上的参数类型是兼容的。 



函数如果你不想指定类型，TypeScript的类型系统会推断出参数类型，因为函数直接赋值给了 SearchFunc类型变量。函数的返回值类型是通过其返回值推断出来的


```js
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```
