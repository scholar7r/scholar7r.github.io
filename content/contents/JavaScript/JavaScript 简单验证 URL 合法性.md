---
title: "JavaScript 简单验证 URL 合法性"
date: 2023-12-10T18:38:07+08:00
categories: JavaScript
draft: false
---
JavaScript 在经过因为多年没有一种简单的方法来验证 URL 之后出现了一种新方法，即为 `URL.canParse()`。

<!--more-->

根据 MDN 的兼容性数据，该方法在 Google Chrome、Edge、Firefox、Safari、Android Webview 中都能良好运行，唯独 Samsung 浏览器还未对其提供支持。

通过这个方法可以对其进行包装，以此来判断 URL 是否合法。

```javascript
function isUrlValid(string) {
    try {
        new URL(string);
        return true;
    } catch (err) {
        return false;
    }
}
```

---

参考：[A new method to validate URLs in JavaScript (2023 edition)](https://www.stefanjudis.com/blog/validate-urls-in-javascript/)
