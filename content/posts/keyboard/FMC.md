---
title: "FMC"
author: "Haobo Gu"
tags: [stm32]
date: 2022-09-18T16:19:23+08:00
summary: 使用FMC驱动8080接口屏幕
---

# 使用FMC驱动8080接口屏幕

## FMC

STM32中的FMC主要是用来控制外接存储。由于8080接口的读写时序和很多外接存储很相似，所以FMC可以用来直接驱动8080接口的屏幕。CubeMX中也提供了相应的功能，非常简单方便。

STM32会把外接RAM映射为4个bank，每个bank对应一个地址区。FMC可以直接写数据到这些地址，而不用在手写复杂的外接RAM时序：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220921233514241.png" alt="image-20220921233514241" style="zoom:50%;" />



## 8080接口

FMC和8080屏幕的接口对应如下：

- FMC D[0:15]：数据线D0~D15
- FMC NEx：片选信号
- FMC NOE：读使能
- FMC NWE：写使能
- FMC Ax：地址线，命令/数据选择

下图是FMC和8080接口的典型连接：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220921233622625.png" alt="image-20220921233622625" style="zoom:50%;" />

## 时序

在配置FMC驱动8080接口屏幕时，时序的配置非常重要。首先我们可以了解下8080接口的时序（一般具体的时序值请查阅驱动芯片的datasheet）：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220921233838818.png" alt="image-20220921233838818" style="zoom:50%;" />

在驱动芯片的datasheet中找到这些值，后面在配置FMC时序时会用到。

在我使用是st7789v驱动芯片中，这些值为：

- t_ah: 10 ns
- t_as: 0 ns
- t_cyc: 66 ns
- t_cyc(read): 160/450 ns（读屏幕RAM/读ID）
- t_wrlw: 15ns
- t_wrlr: 355 ns/45 ns（读屏幕RAM/读ID）
- t_wrhw: 15 ns
- t_wrhr: 90 ns
- t_ds: 10 ns
- t_dh: 10ns
- t_acc: < 340ns/40ns（读屏幕RAM/读ID）。t_acc：10ns对于读来说
- t_od: 20~80ns

根据这些时间，另外可以在CubeMX中，看到FMC运行的时钟频率（如280MHZ），那么一个tick就是3.57ns。根据这些参数，我们就可以去设置FMC的时间参数。计算方法如下：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220922002254176.png" alt="image-20220922002254176" style="zoom:40%;" />

当然这里是针对F103的FSMC。对于我们用的h7b0 + st7789v的组合，经过我好长时间的调试，发现应该在代码里面这么设置：

```c
  /* Timing */
  Timing.AddressSetupTime = 0;
  Timing.AddressHoldTime = 0;
  Timing.DataSetupTime = 18;
  Timing.BusTurnAroundDuration = 0;
  Timing.CLKDivision = 2;
  Timing.DataLatency = 2;
  Timing.AccessMode = FMC_ACCESS_MODE_A;
  /* ExtTiming */
  ExtTiming.AddressSetupTime = 0;
  ExtTiming.AddressHoldTime = 0;
  ExtTiming.DataSetupTime = 10;
  ExtTiming.BusTurnAroundDuration = 0;
  ExtTiming.CLKDivision = 2;
  ExtTiming.DataLatency = 2;
  ExtTiming.AccessMode = FMC_ACCESS_MODE_A;
```

其中，`Timing`是读时序，而`ExtTiming`是写时序。

