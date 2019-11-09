---
title: 栈引发的问题思考
date: 2019-11-09 11:02:14
tags: JavaScript
---
{% asset_img banner.png 图片 %}

<!-- more -->
## 栈的方法

ECMAScript数组也提供了一种让数组的行为类似于其他数据结构的方法。具体说来，数组可以表现得就像栈一样，后者是一种可以限制插入和删除项的数据结构。



栈是一种LIFO（Last-In-First-Out，后进先出）的数据结构，也就是最新添加的项最早被移除。而栈中项的插入（叫做推入）和移除（叫做弹出），只发生在一个位置——栈的顶部。ECMAScript为数组专门提供了 push() 和 pop() 方法，以便实现类似栈的行为。


{% asset_img push.jpg 图片 %}



push() 方法可以接收任意数量的参数，把它们逐个添加到数组末尾，并返回修改后数组的长度。而 pop() 方法则从数组末尾移除最后一项，减少数组的 length 值，然后返回移除的项。



## 栈的应用




可以利用栈将一个数字从一种数制转换成另一种数制。假设想将数字 n 转换为以 b 为基数的数字，实现转换的算法如下。



(1) 最高位为 n % b，将此位推入栈。
(2) 使用 n/b 代替 n。
(3) 重复步骤 1 和 2，直到 n 等于 0，且没有余数。
(4) 持续将栈内元素弹出，直到栈为空，依次将这些元素排列，就得到转换后数字的字符串形式。



使用栈，在 JavaScript 中实现该算法就是小菜一碟。下面就是该函数的定义，可以将数字转化为二至九进制的数字：


```js
function mulBase(num, base) {
  let list = []
  do {
    list.push(num % base)
    num = Math.floor(num /= base)
  } while (num > 0)
  let converted = ''
  while (list.length > 0) {
    converted += list.pop()
  }
  return converted
}

let num = 100
console.log(mulBase(num, 2)) // 1100100
console.log(num.toString(2)) // 1100100
```

回文是指这样一种现象：一个单词、短语或数字，从前往后写和从后往前写都是一样的。比如，单词“dad”、“racecar”就是回文；如果忽略空格和标点符号，下面这个句子也是回文，“A man, a plan, a canal: Panama”；数字 1001 也是回文。



使用栈，可以轻松判断一个字符串是否是回文。我们将拿到的字符串的每个字符按从左至右的顺序推入栈。当字符串中的字符都入栈后，栈内就保存了一个反转后的字符串，最后的字符在栈顶，第一个字符在栈底。



字符串完整压入栈内后，通过持续弹出栈中的每个字母就可以得到一个新字符串，该字符串刚好与原来的字符串顺序相反。我们只需要比较这两个字符串即可，如果它们相等，就是一个回文。


```js
function isPalindrome(word) {
  let list = []
  for(let i = 0; i < word.length; i++) {
    list.push(word[i])
  }
  let rword = ''
  while (list.length > 0) {
    rword += list.pop()
  }
  return word === rword
}
```

为了演示如何用栈实现递归，考虑一下求阶乘函数的递归定义。首先看看 5 的阶乘是怎么定义的。


使用栈来模拟计算 5! 的过程，首先将数字从 5 到 1 推入栈，然后使用一个循环，将数字挨个弹出连乘，就得到了正确的答案：120。

```js
function fact(n) {
  let list = []
  while (n > 1) {
    list.push(n--);
  }
  let product = 1
  while (list.length > 0) {
    product *= list.pop()
  }
  return product
}

console.log(fact(5)) // 150
```


数据结构是计算机存储、组织数据的方式。数据结构是指相互之间存在一种或多种特定关系的数据元素的集合。通常情况下，精心选择的数据结构可以带来更高的运行或者存储效率。

——《基本概念》



## 提问


栈可以用来判断一个算术表达式中的括号是否匹配。编写一个函数，该函数接受一个算术表达式作为参数，返回括号缺失的位置。下面是一个括号不匹配的算术表达式的例子：


```
2.3 + 23 / 12 + (3.14159×0.24
```
