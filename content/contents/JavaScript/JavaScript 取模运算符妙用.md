---
title: "JavaScript 取模运算符妙用"
date: 2023-11-24T22:32:25+08:00
categories: JavaScript
draft: false
---
几乎每种编程语言都提供了使用百分号 `%` 求余数的运算方式，即取模。这种运算方式在实际的开发中貌似很少被用到，或者说，它的身影更多的出现在「判断概改年份是平年还是闰年」。作为一个开发者而不是数学家，该运算符在实际运用中有如何价值？

<!--more-->

将这个想法代入到一个场景中，假设当前有一个包含三个颜色的数组，程序每秒都会获取这三个颜色中的下一个颜色，当达到颜色数组末尾的时候又跳回第一项，那么在一般情况下会写下这样的代码。

```javascript
const COLORS = ['red', 'yellow', 'blue'];

getColor({ timeElapsed: 0 }); // 'red'
getColor({ timeElapsed: 1 }); // 'yellow'
getColor({ timeElapsed: 2 }); // 'blue'
getColor({ timeElapsed: 3 }); // 'red'
getColor({ timeElapsed: 4 }); // 'yellow'
getColor({ timeElapsed: 5 }); // 'blue'
```

`getColor` 会随着时间的不断推移反复执行，每次执行函数本身都会依次返回一个颜色，如果要用代码去实现这个方法，貌似有点麻烦，此时利用取模运算符即可快速实现「根据时间推移依次获取数组中的颜色」。

```javascript
const COLORS = ['red', 'yellow', 'blue'];

function getColor({ timeElapsed }) {
    const colorIndex = timeElapsed % COLORS.length;

    return COLORS[colorIndex];
}
```

`COLORS` 数组中共计 3 中颜色，保持这段代码不变，那么对 `colorIndex` 变量的赋值运算将如此成立 `timeElapsed % 3`。模拟 `getColor` 函数的执行过程，会发现无论 `timeElapsed` 变量变化，结果都始终在数组的可引用范围之内。

```javascript
const colorIndex = 0 % 3; // 0
const colorIndex = 1 % 3; // 1
const colorIndex = 2 % 3; // 2
const colorIndex = 3 % 3; // 0
const colorIndex = 4 % 3; // 1
const colorIndex = 5 % 3; // 2
const colorIndex = 6 % 3; // 0
const colorIndex = 7 % 3; // 1
const colorIndex = 8 % 3; // 2
```

---
参考：[Understanding the JavaScript Modulo Operator (joshwcomeau.com)](https://www.joshwcomeau.com/javascript/modulo-operator/)

