---
title: "Learn QMK - 3"
author: "Haobo Gu"
tags: [keyboard, qmk]
date: 2022-11-8T15:38:55+08:00
summary: Porting new MCUs to QMK
draft: true
---
# 移植QMK

要想移植新的MCU到QMK中，可以首先参考已有的几个PR：
- https://github.com/qmk/qmk_firmware/pull/16529
- https://github.com/qmk/qmk_firmware/pull/16016

## 步骤
下面记录一下都需要哪些步骤。

算了，需要修改chibiOS，太慢了。
直接参考amk，自己写固件，使用ThreadX和vial。
