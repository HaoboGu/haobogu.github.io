---
title: "使用VSCode + RTT方便地调试STM32"
author: "Haobo Gu"
tags: [keyboard, embedded]
date: 2023-04-08T22:11:15+08:00
summary: Print logs using RTT on VSCode
---

一直以来，我都用的是openocd + vscode来开发stm32，相比传统的KEIL MDK，VSCode无论是各种插件（Copilot！）还是人性化方面都更胜一筹。用习惯之后，不论是Python还是Rust还是C，所有的语言都在一个地方开发，真的很爽。在开发stm32的时候，由于引入了RTOS，所以仅仅是断点单步调试显得有些不够用了，就想着可不可以像其他高级语言一样，电脑端命令行中实时打印日志。几番搜索下来，发现VSCode上面的这套开源工具链（OpenOCD + CortexDebug）已经把这个事情做了！先展示一下效果：

![image-20230408224147062](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20230408224147062.png)

单步调试 + 彩色日志，直接IDE内命令行展示，全齐，且不需要额外硬件，一个stlink足够。真的很方便。下面就简单介绍一下如何实现。

## 安装RTT

首先是要从SEGGER网站上面下载JLink全家桶，并且把RTT相关代码复制到工程下面，在makefile中添加对应的文件。参见：https://zhuanlan.zhihu.com/p/163771273 这篇文章。其中，如果你要使用stlink，那么环境变量也不用配置，只需要复制RTT源码即可。然后，添加log文件，同样是从上面的文章中复制得到，感谢文章作者：

```c
/*
 * Author: Jayant Tang
 * Email: jayant97@foxmail.com
 */

#ifndef _LOG_H_
#define _LOH_H_
#include "SEGGER_RTT.h"

#define LOG_DEBUG 1

#if LOG_DEBUG


#define LOG_PROTO(type,color,format,...)            \
        SEGGER_RTT_printf(0,"  %s%s"format"\r\n%s", \
                          color,                    \
                          type,                     \
                          ##__VA_ARGS__,            \
                          RTT_CTRL_RESET)

/* 清屏*/
#define LOG_CLEAR() SEGGER_RTT_WriteString(0, "  "RTT_CTRL_CLEAR)

/* 无颜色日志输出 */
#define LOG(format,...) LOG_PROTO("","",format,##__VA_ARGS__)

/* 有颜色格式日志输出 */
#define LOGI(format,...) LOG_PROTO("INFO: ", RTT_CTRL_TEXT_BRIGHT_GREEN , format, ##__VA_ARGS__)
#define LOGW(format,...) LOG_PROTO("WARN: ", RTT_CTRL_TEXT_BRIGHT_YELLOW, format, ##__VA_ARGS__)
#define LOGE(format,...) LOG_PROTO("ERROR: ", RTT_CTRL_TEXT_BRIGHT_RED   , format, ##__VA_ARGS__)

#else
#define LOG_CLEAR()
#define LOG
#define LOGI
#define LOGW
#define LOGE

#endif

#endif // !_LOG_H_
```

## 配置Cortex-Debug

OpenOCD原生是支持RTT的，在默认情况下需要去修改`openocd.cfg`来配置RTT。不过在VSCode里面，Cortex-Debug插件已经帮你配置好了一切（当然底层还是使用的OpenOCD的能力）！本来Cortex-Debug也是在VSCode下调试stm32必装的软件，所以说我们实际上只需要改改配置就可以使用RTT了。

具体Cortex-Debug的配置可以参见我之前的文章：https://haobogu.github.io/posts/keyboard/openocd-ospi-flash/ 中的`launch.json`配置环节。

在这里，我们只需要在原本的`launch.json`中，增加如下配置：

![image-20230408225111211](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20230408225111211.png)

然后，按F5烧录代码并启动调试。你就会在下面的终端页面上看到新增了一个窗口：

![image-20230408225245588](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20230408225245588.png)

这里就是RTT日志显示的地方。这样，不管是单步调试还是彩色日志显示，在一个地方全齐。Cortex-Debug甚至还支持RTT的其他特性，比如Graph：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20230408230952997.png" alt="image-20230408230952997" style="zoom:50%;" />

更多的使用方法，可以参考Cortex-Debug的WIKI：https://github.com/Marus/cortex-debug/wiki/SEGGER-RTT-support

