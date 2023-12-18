---
title: "Lvgl"
author: "Haobo Gu"
tags: [keyboard, stm32, gui]
date: 2022-09-18T00:03:23+08:00
summary: 学习Lvgl的笔记
draft: true
---

# LVGL
## Import Lvgl

得使用`lv_conf.c`配置。

## Lvgl Development



## 显示GIF

踩坑记录

如果只显示一个白色的背景，那么很有可能是因为内存不够了。尝试增加内存：

```c
# lv_conf.h
#define LV_MEM_CUSTOM 0
#if LV_MEM_CUSTOM == 0
    /*Size of the memory available for `lv_mem_alloc()` in bytes (>= 2kB)*/
    // 在这里增加内存
    #define LV_MEM_SIZE (160U * 1024U)          /*[bytes]*/

    /*Set an address for the memory pool instead of allocating it as a normal array. Can be in external SRAM too.*/
    // 对于h7b0，默认情况下使用的是DTCMRAM，直接增加大概率会显示内存不够。此时可以把内存地址改为RAM，即0x24000000
    #define LV_MEM_ADR 0x24000000     /*0: unused*/
    /*Instead of an address give a memory allocator that will be called to get a memory pool for LVGL. E.g. my_malloc*/
    #if LV_MEM_ADR == 0
        #undef LV_MEM_POOL_INCLUDE
        #undef LV_MEM_POOL_ALLOC
    #endif

```



## ffmpeg

可以使用ffmpeg对gif做转换。

下面是几个常用的选项：

首先是放在 `-i`前面的，即对输入做处理：

- `-t`：duration，时间长度，单位是秒。
- `-ss`： 开始的时间戳，可以用一个数字，比如`3`表示从第三秒开始。也可以用`hh:mm:ss`格式，如`00:00:03`。

下面是一个例子，截取从第一秒到第三秒共2秒时间：

```shell
ffmpeg -t 2 -ss 1 -i input.mp4 -o output.mp4
```

然后，是放在`-o`前面的，可以对输出的文件做一定的处理：

- `-r`：设置帧率
- `-vf`：设置filter，可以通过参数设置帧率和缩放等信息

转换任意格式到gif，然后设置高度为100，宽度自适应，代码如下：

