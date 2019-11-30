---
title: typescript接口使用
date: 2019-11-30 11:52:07
tags: TypeScript
---

{% asset_img banner.png 图片 %}

TypeScript的核心原则之一是对值所具有的结构进行类型检查。  在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。
<!-- more -->


下面通过一个简单示例来观察接口是如何工作的：
```ts
function printLabel(labelObj: { label: string }) {
  console.log(labelObj.label);
}
let myObj = { size: 10, label: 'Size 10 Object' }
printLabel(myObj)
```

类型检查器会查看printLabel的调用。printLabel有一个参数，并要求这个对象参数有一个名为label类型为string的属性。使用到了的属性必须要定义类型，否则会报错。
```ts
function printLabel(labelledObj: { label: string }) {
  // 类型“{ label: string; }”上不存在属性“size”
  console.log(labelledObj.size); // Error
}
```

```ts
let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```


需要注意的是，我们传入的对象参数实际上会包含很多属性，但是编译器只会检查那些必需的属性是否存在，并且其类型是否匹配。 
```ts
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}
```
// 类型“{ title: string; }”的参数不能赋给类型“{ label: string; }”的参数。
```ts
let myObj = { title: 'hello' };
printLabel(myObj); // Error
```


下面我们重写上面的例子，这次使用接口来描述：必须包含一个label属性且类型为string：
```ts
interface LabelledValue {
  label: string;
}
function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}
```

```ts
let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```


LabelledValue接口就好比一个名字，用来描述上面例子里的要求。它代表了有一个 label属性且类型为string的对象。还有一点值得提的是，类型检查器不会去检查属性的顺序，只要相应的属性存在并且类型也是对的就可以。



## 可选属性

接口里的属性不全都是必需的。有些是只在某些条件下存在，或者根本不存在。
```ts
interface SquareConfig {
  color?: string;
  width?: number;
}
```


带有可选属性的接口与普通的接口定义差不多，只是在可选属性名字定义的后面加一个?符号。



## 只读属性

一些对象属性只能在对象刚刚创建的时候修改其值。你可以在属性名前用 readonly来指定只读属性:
```ts
interface Point {
  readonly x: number;
  readonly y: number;
}
```


你可以通过赋值一个对象字面量来构造一个Point。赋值后， x和y再也不能被改变了。
```ts
let p1: Point = { x: 10, y: 20 }; 
p1.x = 5; // Error
```


## 额外的属性检查

如果一个对象字面量存在任何“目标类型”不包含的属性时，你会得到一个错误。
```ts
interface SquareConfig {
  color?: string;
  width?: number;
}
```


// “height”不在类型“SquareConfig”中。
```ts
let obj:SquareConfig = {
  color: 'red',
  width: 100,
  height: 100 // Error
}
```


如果 SquareConfig 带有上面定义的类型的color和width属性，并且还会带有任意数量的其它属性，那么我们可以这样定义它：
```ts
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}
```
