---
title: TypeScript基础类型
date: 2019-11-30 11:47:03
tags: TypeScript
---

{% asset_img banner.png 图片 %}

Typescript是由微软开发的一款开源的编程语言，是Javascript的超集，TS提供的类型系统可以帮助我们在写代码的时候提供更丰富的语法提示，让我们使用TypeScript来创建一个简单的Web应用。

<!-- more -->


通过npm（Node.js包管理器）安装TypeScript
```
npm install -g typescript
```


构建你的第一个TypeScript文件，在编辑器，将下面的代码输入到 greeter.ts 文件里：
```ts
let isDone: boolean = false
```


我们使用了.ts扩展名，在命令行上，运行TypeScript编译器：
```
tsc greeter.ts
```


输出结果为一个greeter.js文件。一切准备就绪，我们可以运行这个使用TypeScript写的JavaScript应用了！



为了让程序有价值，我们需要能够处理最简单的数据单元：数字，字符串，结构体，布尔值等。TypeScript支持与JavaScript几乎相同的数据类型，此外还提供了实用的枚举类型方便我们使用。



最基本的数据类型就是简单的true/false值，在JavaScript和TypeScript里叫做boolean。
```ts
let isDone: boolean = false;
```

和JavaScript一样，TypeScript里的所有数字都是浮点数。这些浮点数的类型是 number。
```ts
let decLiteral: number = 6;
```


我们使用 string表示文本数据类型。 
```ts
let name: string = "bob";
```


TypeScript像JavaScript一样可以操作数组元素。有两种方式可以定义数组。第一种，可以在元素类型后面接上[]，表示由此类型元素组成的一个数组：
```ts
let list: number[] = [1, 2, 3];
```


第二种方式是使用数组泛型，Array<元素类型>：
```ts
let list: Array<number> = [1, 2, 3];
```


元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。比如，你可以定义一对值分别为string和number类型的元组。
```ts
let x: [string, number];
x = ['hello', 10]; // OK
x = [10, 'hello']; // Error
```


枚举enum类型是对JavaScript标准数据类型的一个补充。像C#等其它语言一样，使用枚举类型可以为一组数值赋予友好的名字
```ts
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```


默认情况下，从0开始为元素编号。你也可以手动的指定成员的数值。例如，我们将上面的例子改成从 1开始编号：
```ts
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green;
```


或者，全部都采用手动赋值：
```ts
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green;
```


枚举类型提供的一个便利是你可以由枚举的值得到它的名字。例如，我们知道数值为2，但是不确定它映射到Color里的哪个名字，我们可以查找相应的名字：
```ts
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2];
console.log(colorName);  // 显示'Green'因为上面代码里它的值是2
```




any类型，有时候，我们会想要为那些在编程阶段还不清楚类型的变量指定一个类型。这些值可能来自于动态的内容，比如来自用户输入或第三方代码库。这种情况下，我们不希望类型检查器对这些值进行检查而是直接让它们通过编译阶段的检查。那么我们可以使用 any类型来标记这些变量：
```ts
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false;
```

void类型，某种程度上来说，void类型像是与any类型相反，它表示没有任何类型。当一个函数没有返回值时，你通常会见到其返回值类型是 void：
```ts
function warnUser(): void {
  console.log("This is my warning message");
}
```


声明一个void类型的变量没有什么大用，因为你只能为它赋予undefined和null：
```ts
let unusable: void = undefined;
```


TypeScript里，undefined和null两者各自有自己的类型分别叫做undefined和null。和 void相似，它们的本身的类型用处不是很大：
```ts
let u: undefined = undefined;
let n: null = null;
```


默认情况下null和undefined是所有类型的子类型。就是说你可以把 null和undefined赋值给number类型的变量。
```ts
let x: number;
x = 1;
x = undefined;
x = null;
```


然而，当你指定了--strictNullChecks标记，null和undefined只能赋值给void和它们各自。通过命令生成配置文件才能配置该参数，该参数是默认是开启的。
```
tsc --init
```


执行命令之后会生成tsconfig.json文件，可以根据用户的习惯进行配置参数。



never类型表示的是那些永不存在的值的类型。例如， never类型是那些总是会抛出异常或根本就不会有返回值的函数表达式或箭头函数表达式的返回值类型；变量也可能是 never类型，当它们被永不为真的类型保护所约束时。



下面是一些返回never类型的函数：

// 返回never的函数必须存在无法达到的终点
```ts
function error(message: string): never {
  throw new Error(message);
}
```


// 推断的返回值类型为never
```ts
function fail() {
  return error("Something failed");
}
```


// 返回never的函数必须存在无法达到的终点
```ts
function infiniteLoop(): never {
  while (true) {
  }
} 
```


object表示非原始类型，也就是除number，string，boolean，symbol，null或undefined之外的类型。



使用object类型，就可以更好的表示像Object.create这样的API。例如：
```ts
declare function create(o: object | null): void;
create({ prop: 0 }); // OK
```


类型断言，有时候你会遇到这样的情况，你会比TypeScript更了解某个值的详细信息。通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型



类型断言有两种形式。其一是“尖括号”语法：
```ts
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
```


另一个为`as`语法：
```ts
let someValue: any = "this is a string"
let strLength: number = (someValue as string).length;
```
