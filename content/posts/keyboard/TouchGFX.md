---
title: "TouchGFX"
author: "Haobo Gu"
tags: [keyboard, stm32]
date: 2022-09-18T00:03:23+08:00
summary: 学习TouchGFX的笔记
---

# TouchGFX

文档链接：https://support.touchgfx.com/4.20/zh-CN/docs/category/introduction

## 学习TouchGFX的步骤

首先，开发基于stm32的TouchGFX应用可以分成四个步骤如下图：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220919222109215.png" alt="image-20220919222109215" style="zoom:50%;" />

前两个步骤硬件选择和开发板配置和TouchGFX关系不大，因此我们直接跳过，从TouchGFX本身的开发开始。首先我们需要学习TouchGFX的抽象层的开发，即TouchGFX AL

## TouchGFX的抽象层（AL）

TouchGFX AL由两个部分组成，分别是硬件抽象层（HAL）和操作系统抽象层（OSAL）。顾名思义，这两个抽象层实际上就充当了TouchGFX和硬件以及操作系统连接的桥梁，把硬件、TouchGFX引擎以及操作系统连接起来，从而构建在单片机上运行的图形界面程序。

### 抽象层的职责

那么，抽象层具体负责哪些事情呢？[这里](https://support.touchgfx.com/4.20/zh-CN/docs/development/touchgfx-hal-development/touchgfx-al-development-introduction#responsibilities-of-the-abstraction-layer)有一个表格，里面列举了抽象层的具体职责。简单来说，就是如下几件事情：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220919223611452.png" alt="image-20220919223611452" style="zoom:33%;" />

在TouchGFX引擎的主循环中，会自动地调用AL层的若干个钩子函数（或者触发中断），从而完成上述的这些事情：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220919223335794.png" alt="image-20220919223335794" style="zoom:33%;" />

我们需要做的就是去开发这些钩子函数，从而搭建起TouchGFX到硬件以及操作系统的桥梁。下面我们会介绍每个钩子的具体职责和用法。

#### 将TouchGFX Engine主循环与显示器传输同步

此步骤背后的主要思想是，**在渲染完成后阻塞TouchGFX Engine主循环，从而确保在显示设备准备好之前不再产生其他帧**。 一旦显示设备准备就绪，OSAL向被阻塞的Engine主循环发出信号，以继续**产生**（生产、渲染）显示帧。

开发者可以通过`Rendering done`钩子以及`Display Ready`中断来完成这个步骤，然后通过使用OSAL中的`OSWrappers::signalVSync`来完成同步。

下面会介绍`Rendering done`和`Display Ready`。

##### 渲染完成（Rendering done）

渲染完成钩子即`OSWrappers::waitForVSync`会在渲染完成之后被TouchGFX引擎自动调用。开发者在实现这个AL方法的时候，**必须阻塞图形引擎**，直到渲染下一帧。标准的实现方式是从一个消息队列中读取，读不到则阻塞。当`OSWrapper::signalVSync`发出信号时，`OSWrappers::waitForVSync`会收到信号并且解除阻塞，然后TouchGFX会开始渲染下一帧。

##### 显示就绪（Display Ready）

`Display Ready`中断应该来自显示屏（或者相关controller、定时器），这个中断会触发主循环停止阻塞并继续渲染下一帧。在中断触发之后，就应该去调用`OSWrappers::signalVsync`来使上个section介绍的`OSWrappers::waitForVsync`解除阻塞。

下面是一个基于RTOS的实现示例，如果使用TouchGFX Generator，RTOS的代码会自动地生成好。其他OS的需要参考这些代码自行实现OSWrappers：

```c++
// RTOS_OSWrappers.cpp

// 
static osMessageQId vsync_queue = 0; //Queue identifier is assigned elsewhere

// 在中断中调用该函数，往vsync_queue中放一个值，waitForVSync收到之后会通知引擎继续渲染下一帧
void OSWrappers::signalVSync()
{
    if (vsync_queue)
    {
        osMessagePut(vsync_queue, dummy, 0);
    }
}

// 在渲染完一帧之后，在这里阻塞，直到收到队列中的值，继续下一帧渲染
void OSWrappers::waitForVSync()
{
    uint32_t dummyGet;
    // First make sure the queue is empty, by trying to remove an element with 0 timeout.
    osMessageQueueGet(vsync_queue, &dummyGet, 0, 0);

    // Then, wait for next VSYNC to occur.
    osMessageQueueGet(vsync_queue, &dummyGet, 0, osWaitForever);
}

```

#### 触摸与其他外部事件

比较简单，看看就行。说白了就两种方式：轮询/中断。

见：https://support.touchgfx.com/4.20/zh-CN/docs/development/touchgfx-hal-development/touchgfx-architecture#report-touch-and-physical-button-events

#### 同步帧缓冲访问

帧缓冲（framebuffer）是整个图形系统要操作的核心数据。在整个显示过程中，会有很多个硬件访问帧缓冲，如CPU、DMA2D、LTDC等，因此TouchGFX AL必须提供一种保护该存储器的方式，从而不让数据读写出现问题（比如CPU写一半，DMA2D就去读取了）。

TouchGFX通过`OSWrappers`中的接口来同步帧缓冲的访问，最常规的实现方式是通过信号量来保护帧缓冲的访问，当然也可以自行实现其他方式。下表定义了`OSWrappers`中对访问帧缓冲需要用到的函数列表，这些函数可以由TouchGFX Generator生成（for RTOS），也可以自行实现：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220919232514202.png" alt="image-20220919232514202" style="zoom: 50%;" />

#### 报告下一个可用的帧缓冲区

TouchGFX必须在每个Tick知道下一个可用的帧缓冲区。帧缓冲的策略可以有多种，使用单帧缓冲或双帧缓冲时，TouchGFX Engine将根据帧缓冲的全宽、高度和位宽将像素数据写入存储区。而对于部分缓存，可以参见[这里](https://support.touchgfx.com/4.20/zh-CN/docs/development/scenarios/lowering-memory-usage-with-partial-framebuffer)。

#### 执行渲染操作

很多stm32的MCU都提供了把内容渲染到帧缓冲的外设，如DMA2D。在渲染内容到帧缓冲之前，TouchGFX引擎会首先检查HAL层是否实现了渲染功能。如果实现了，那么这个操作就会直接交给HAL层完成，否则会交给CPU处理。

TouchGFX引擎会调用`HAL::getBlitCaps()`来获取硬件的能力描述，开发者需要实现对应的HAL子类来把MCU的硬件渲染能力添加进来。然后，TouchGFX引擎会使用HAL类中定义的具体操作，如`HAL::blitCopy()`等，去执行具体的渲染工作。如果HAL没有实现对应的函数，TouchGFX会默认使用CPU完成这些事情。

#### 把帧缓冲传输到显示设备

每当一部分帧缓冲渲染完成之后，TouchGFX引擎会调用`Rendering of area complete`钩子来通知AL层去把渲染完成的那部分帧缓冲传输到显示设备。

##### Rendering of area complete钩子

在代码里面，这个钩子是一个虚函数：`HAL::flushFrameBuffer(Rect& rect)`。在LTDC中我们不用管这个东西，因为LTDC会自动把这些事情做了。这个函数留空即可。而对于其他的接口，如SPI/8080，则需要开发者手动实现该函数实现对帧缓冲的传输。

对于带GRAM的显示设备（即显示设备里面也有RAM，并口屏常见），这个函数的实现允许手动发起向GRAM的帧缓冲区域的传输：

```c++
void TouchGFXHAL::flushFrameBuffer(const touchgfx::Rect& r)
{
    HAL::flushFrameBuffer(rect); //call superclass

    //start transfer if not running already!
    if (!IsTransmittingData())
    {
        const uint8_t* pixels = ...; // Calculate pixel address
        SendFrameBufferRect((uint8_t*)pixels, r.x, r.y, r.width, r.height);
    }
    else
    {
       ... // Queue rect for later or wait here
    }
}
```

### TouchGFX的硬件抽象层（HAL）

了解了抽象层各个钩子的职责之后，首先看一下硬件抽象层（HAL）。

## TouchGFX Generator

TouchGFX Generator是开发TouchGFX应用的必备工具，和CubeMX类似，TouchGFX Generator是用来生成TouchGFX代码的工具，里面提供了有关显示的各种配置。和CubeMX不同的是，TouchGFX生成的是C++代码，通过继承向开发者提供灵活性。

在CubeMX里面开启TouchGFX Generator之后，生成的代码会自动创建一个TouchGFX文件夹并且把相关代码文件都生成进去。下面是TouchGFX项目的项目结构：

```
│   .mxproject
│   myproject.ioc
├───Core
├───Drivers
├───EWARM
├───Middlewares
└───TouchGFX
    │   ApplicationTemplate.touchgfx.part
    ├───App
    │       app_touchgfx.c
    │       app_touchgfx.h
    └───target
        │   STM32TouchController.cpp
        │   STM32TouchController.hpp
        │   TouchGFXGPIO.cpp
        │   TouchGFXHAL.cpp
        │   TouchGFXHAL.hpp
        │
        └───generated
                OSWrappers.cpp
                TouchGFXConfiguration.cpp
                TouchGFXGeneratedHAL.cpp
                TouchGFXGeneratedHAL.hpp
```

在这里我们只关心TouchGFX文件夹下的内容：

- ApplicationTemplate.touchgfx.part：TouchGFX Designer工程相关的一些配置，比如屏幕尺寸、位深等
- App：X-Cube接口，其中`app_touchgfx.c`包含`MX_TouchGFX_Process(void)`和`MX_TouchGFX_Init(void)`函数，这些函数用于在CubeMX生成的主函数中初始化TouchGFX以及开始主循环
- target/generated：注意这个目录是**只读的**，包含根据相关配置生成的TouchGFX源文件。简单来说，`TouchGFXGeneratedHAL`是自动生成的HAL层，`OSWrappers`是OSAL层。而`TouchGFXConfiguration`是TouchGFX的相关配置，包含**一个用于构建HAL的函数**以及**启动TouchGFX主函数的函数**。

- target：这个目录是开发者的代码所在，可以继承`target/generated`里面的类从而实现对HAL/OSAL的覆写。也可以拓展HAL，添加新功能等等。生成的`STM32TouchController.cpp`文件包含了触摸控制的空接口，而`TouchGFXHAL.cpp`定义了`TouchGFXGeneratedHAL`的子类。

需要注意的是，对于HAL来说，`TouchGFXHAL`会继承`TouchGFXGeneratedHAL`，用户可能需要修改`TouchGFXHAL`来完成HAL的其他配置。HAL的一般架构如下所示：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220920175830984.png" alt="image-20220920175830984" style="zoom:50%;" />



