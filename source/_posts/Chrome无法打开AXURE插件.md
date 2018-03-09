---
title: Chrome无法打开AXURE插件解决办法
description:  Chrome无法打开AXURE插件解决办法
date: 2018-03-08 20:40:00
comments: true
tags: 
    - chrome
    - 黑科技  
categories:
    - 黑科技
---

## 问题
打开原型网页显示如下

![图片](/images/axure.jpg)

## 解决方法
注释原型index.html中以下代码
```js
$(window).bind('load', function() {
    if(CHROME_5_LOCAL && !$('body').attr('pluginDetected')) {
        window.location = 'resources/chrome/chrome.html';
    }
});
```