---
title: "Keyboard Design"
author: "Haobo Gu"
tags: [keyboard]
date: 2022-09-01T16:16:08.734127+08:00
draft: true
summary: Design your own keyboard
---

# Design

## Generate Plate

1. open builder.swillkb.com
2. choose switch type, use MX{_t:1} in general
3. choose stablizer type, use cherry only {_s:2}
4. 



## 程序设计

主要包括如下几个部分

1. bootloader
2. 键盘固件程序
3. 用户可以自定义的图片
4. 其他UI资源，如图片、字体

现在的方案是，bootloader在内置rom，固件在外置SPI，同时用户自定义图片也在外置SPI的后4M。

这样的话，内置的UI资源就没有地方放了（或者，放到2-4M的位置，固件2M or 放到内置ROM）

这样的话一共8M的SPI Flash就够用了。

我jo得可行。

