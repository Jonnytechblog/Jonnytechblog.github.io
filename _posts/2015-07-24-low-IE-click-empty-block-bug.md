---
layout: post
title:  "在低版本 IE 中点击空 block 元素的问题"
date:   2015-07-24 00:06:05
categories: CSS
excerpt: 低版本IE的bug和兼容性
---

* content
{:toc}

## �

�决方法很简单，即给这个块级元素填充任意颜色，然后将其透明度设置为0。代码如下：

    background-color: #fff;
    opacity: 0;
    filter:alpha(opacity=0);
