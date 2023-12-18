---
title: "STM32"
author: "Haobo Gu"
tags: [keyboard]
date: 2022-07-30T15:23:40+08:00
summary: 学习STM32的笔记
---

# STM32

学习STM32的笔记，视频教程：https://www.bilibili.com/video/BV1th411z7sn?p=2&spm_id_from=pageDriver&vd_source=58fb33df0449f8258f0e273447aab712

## 片上资源

首先，需要了解一下STM32芯片上面都有哪些可以使用的资源，如下图：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730152453142.png" alt="image-20220730152453142" style="zoom:50%;" />

简单记忆一下各种资源/外设的缩写即可，后面会详细用到

## 引脚定义

引脚定义是学习一个单片机的核心，一般来说如果了解了引脚定义，这个单片机怎么用的基本就知道个八九不离十了。例子里面是STM32F103C8T6， 详细定义可以见下面的表。这里首先讲一下每个引脚的含义：

- VBAT： 备用电源
- PC13-TAMPER-RTC：这个引脚有多重功能，分别是PC13（IO口）、TAMPER（侵入检测）、RTC（输出RTC校准时钟、秒脉冲等）

- PC14-OSC32_IN：IO或接32KHZ的RTC晶振的IN

- PC15-OSC32_OUT：同上，只不过这个引脚是RTC晶振的OUT

- OSC_IN和OSC_OUT：系统主晶振，一般是8MHZ（芯片里面会对这个频率进行倍频，比如生成72MHZ的频率作为芯片的主时钟）

- NRST：系统复位，N代表是低电平复位

- VSSA和VDDA：芯片内模拟部分的电源，VSS是负极（接GND），VDD正极（接3.3V）

- PAx、PBx和PCx：都是IO，其中有一些特殊的或者有其他功能的，下面会详细讲。注意在表里面没有加粗的IO，都是不推荐作为IO使用的（因为这些IO有其他功能，除非IO实在是不够了，否则还是用那些加粗的IO）

  - PA0-WKUP意味着PA0除了是IO之外，还有wakeup的功能

  - PB2：看主功能栏，这个引脚除了IO之外还有BOOT的功能，即可以用来配置启动模式

  - PA13/14/15、PB3/4：还可以作为调试端口。可以看到有两种调试模式：JTAG和SWD

    > SWD调试需要两根线：SWDIO和SWCLK
    >
    > JTAG则需要5根线：JTMS、JTCK、JTDI、JTDO、NJTRST

  - STLINK使用的SWD，因此只需要两根线。不过这5个IO，默认都不会被用来做普通IO。如果你是用SWD，且想使用剩下的三个IO，那么需要在程序里面配置。

- VSS_x和VDD_x：系统的主电源，同样，VSS负极，VDD正极。由于STM32是分区供电，因此可以有多个电源，这些电源正极/负极可以短接在一起，然后一起供电

- BOOT0：和PB2/BOOT1一样，也是用来做启动配置

![](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730153247643.png)



 ## 启动配置

看完引脚图，大概有个印象即可。下面看一下启动配置。还记得之前表里的两个BOOT吗？BOOT0和PB2/BOOT1。这两个引脚可以用来设置模式：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730155932890.png" alt="image-20220730155932890" style="zoom:35%;" />

引脚为0就是接地GND的意思，1是高电平。默认都是使用flash作为启动区域。另外，下面那句话的意思就是，BOOT0和BOOT1只在刚开始上电之后的一瞬间（第四个上升沿）有用，后面就随便了。

## 最小系统电路

最小系统电路就是能让芯片跑起来的最小电路。下面就看一下F103的最小电路：

### 供电

首先看供电的部分。可以看到，所有的VDD都接了3.3V，然后VSS都接了GND。在所有的3.3V和GND之间都有一个电容，俗称滤波电容。滤波电容主要是保持供电电压的稳定，一般供电最好都来一个比较好。

另外，VBAT是备用的电源，如果需要用到就接，不用到就接3.3V或者悬空。如果用到VBAT，那就接一个3V的纽扣电池即可，电池正极VBAT，负极接地。

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730162453379.png" alt="image-20220730162453379" style="zoom:50%;" />

可以看到，整个供电还是很简单的。但是由于供电口比较多，所以走线可能有些麻烦。

### 晶振

STM32的主晶振一般都是8MHZ。晶振就连接芯片上的OSC_IN和OSC_OUT：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730163244776.png" alt="image-20220730163244776" style="zoom:67%;" />

这两个电容是启震电容，另一端接地即可。

### 复位电路

复位电路的原理稍微麻烦点。在上电的**一瞬间**，电容是相当于短路的，因此就产生了一瞬间的低电平。芯片是低电平复位，所以可以理解为开机的一瞬间复位。之后，电容充电断开，在K1开关断开的情况下，NRST就和3.3V相连，一直是高电平不复位。按下复位开关，NRST接地，就完成了手动复位。

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730163502567.png" alt="image-20220730163502567" style="zoom:50%;" />

### 启动配置电路

就是上面选的BOOT0/1选择启动配置的电路。在我们的411板子上，是这样的：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730212003630.png" alt="image-20220730212003630" style="zoom:50%;" />

可以看到这是一个拨码开关，默认情况下开关处在123的位置，而456这一侧为ON，即默认情况下三个位置都没有接通。从图上可以看到，如果左右没有接通，则三个位置都断开。即BOOT0=0，而PB2/BOOT1悬空。

如果想把BOOT1设置为1，则需要同时把1-6和2-5连接起来，即上面两个开关拨向ON。把BOOT0置1则只需要拨动3-4开关到ON即可。

 ## Hello World工程

下面我们就来写一个最简单的工程来测试。首先我们的板子上，有一个PC13连接的LED，这个就是我们用来测试板子的灯，电路图在这里：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730214406025.png" alt="image-20220730214406025" style="zoom:50%;" />

可以看到其中一个是常亮的pwr指示灯，另外一个是连接PC13端口的测试灯。

然后就是创建工程的步骤了，我们使用VSCode + PlatformIO + CubeMX创建我们的工程，具体可以参考这里：https://haobogu.github.io/posts/develop-stm32-using-vscode/。

OK，创建完工程之后，就可以开始写代码了。打开`main.c`文件，找到主函数里面的`while(1)`循环，发现里面什么都没有。因为默认生成的是空工程。这个时候如果编译下载之后，主板上的绿灯就直接灭了，因为我们啥都没写。那么我们第一个程序就从这里开始。

### 使用寄存器的方式点亮LED

STM32的编程一般有3种方式：

1. 寄存器编程：直接控制芯片中的寄存器，比较基础，速度快。但是由于STM32系列很复杂，寄存器非常多，因此这种方法后期比较麻烦
2. 库函数编程：ST公司提供了可以直接对芯片做各种操作的库函数，可以理解为寄存器操作的封装，十分方便，可以作为主要的编程方式
3. HAL编程：可以理解为低代码编程，通过ST公司的对应软件里面的图形化界面完成编程。但是这种方式不利于对芯片底层的理解，不好开发进阶功能

综上，我们主要会选择库函数编程的方式。当然寄存器编程也要了解。因此我们的第一个程序就从寄存器编程开始（这个章节如果看不懂，无所谓，就了解下）。

#### 寄存器配置

首先是RCC寄存器的时钟，这个可以理解为是GPIO的使能时钟。这里，出现了STM32F4和F1的不同：在F1中GPIO都是APB2的外设，而在F4中，GPIO是AHB1的外设。具体可以看下F411的手册：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730221626472.png" alt="image-20220730221626472" style="zoom:50%;" />

可以看到在F411中，这些GPIO都是和AHB1相连的。这也是和视频中的不一样的地方。不过原理都是相同的，找到对应的使能时钟的RCC寄存器：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730221900311.png" alt="image-20220730221900311" style="zoom:50%;" />

可以看到和视频中类似，有一堆GPIOxEN：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220730221932083.png" alt="image-20220730221932083" style="zoom:50%;" />

我们要操作PC13上面的灯，那就GPIOCEN写入1即可：

```c
// main.c中的主函数中，其他代码省略

// 首先打开GPIOC的时钟
// 由于写入的是16进制，我们需要每4位转换一次，即
RCC -> AHB1ENR = 0x00000004;
```

然后找到对应IO的通用配置高寄存器`GPIOx_CRH`，可以看到对应GPIOC有两个地方，CNF配置为00，即推挽模式，而MODE配置为11（这里F411还是和视频中不一样，我直接写F411的代码）

```c
// 配置输出寄存器  
GPIOC -> MODER = 0x0C000000;
GPIOC -> OTYPER = 0x00000000;	
```

#### 往PC13端口，写入高电平

写出数据的寄存器叫`GPIOx_ODR`，我们往PC13写出高电平，即：

```
GPIOC -> ODR = 0x00002000;   
```

OK，这就是寄存器编程的大概流程。我们在这里主要是简单了解一下编程的流程，即使能时钟、配置IO、输入输出。后面我们会使用HAL库来对STM32进行编程。





