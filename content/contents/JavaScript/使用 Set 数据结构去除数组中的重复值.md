---
title: "使用 Set 数据结构去除数组中的重复值"
date: 2023-10-22T00:40:23+08:00
categories: JavaScript
draft: false
---
Set 是 JavaScript 中的一个数据结构，它只能包含唯一的值，所以可以使用 Set 集合来跟踪不同的数字。

<!--more-->

```javascript
function countUniqueOccrrences(duplicateValues) {
    const uniqueSet = new Set();
    
    for (const value of duplicateValues) {
        uniqueSet.add(value);
    }
    
    return uniqueSet;
}
```

实例中使用 uniqueSet 变量来保存传入的不同数字，通过遍历 duplicateValues 数组，使用 Set 的 add 方法将不同的数字添加的 uniqueSet 集合中。
