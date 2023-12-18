---
title: "使用STM32中的内置flash作为eeprom"
author: "Haobo Gu"
tags: [keyboard, embedded, stm32]
date: 2023-02-26T17:59:37+08:00
summary: 使用STM32中的内置flash作为eeprom
draft: true
---

# 使用STM32中的内置flash作为eeprom

首先参考ST官方的h7 eeprom emulation示例：https://github.com/STMicroelectronics/STM32CubeH7/blob/43c9e552ba1c038577c48723d96ca8c825b11987/Projects/NUCLEO-H7A3ZI-Q/Applications/EEPROM/EEPROM_Emulation/Src/eeprom.c

在官方示例中，使用了两个sector，互相备份。每个sector的前128bits作为header，存储sector的状态信息。

另外需要注意的是，在flash擦除之后，需要invalidate data cache，来保证数据的一致性。

![image-20230226175621559](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20230226175621559.png)

