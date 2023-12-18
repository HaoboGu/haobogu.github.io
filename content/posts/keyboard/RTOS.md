---
title: "RTOS"
author: "Haobo Gu"
tags: [keyboard, embedded]
date: 2022-11-11T23:59:37+08:00
summary: Learn RTOS
---

# RTOS
## 什么是RTOS
RTOS指实时操作系统（Real-Time Operation System）。是广泛用在嵌入式中的操作系统，其主要的特征是基于优先级的任务执行，以及快速、实时的响应。

## Azure RTOS
Azure RTOS的前身是ThreadX，是微软收购并且开源的一个RTOS。Azure RTOS包含一整套的嵌入式系统，实现了全家桶式的嵌入式RTOS解决方案。Azure RTOS主要包括如下组件：

- ThreadX: 高性能实时操作系统
- NetX: 嵌入式网络协议栈
- FileX: 嵌入式FAT文件系统
- LevelX: NAND 和 NOR 闪存磨损均衡
- GuiX: 嵌入式图形库
- USBX: 嵌入式USB协议栈
- TraceX: 基于主机的嵌入式分析工具

RTOS有很多，为什么我会选择Azure RTOS作为我学习使用的对象呢？原因主要有如下三点：

1. 我主要使用STM32，ST官方支持比较好的RTOS只有FreeRTOS和AzureRTOS（ThreadX）
2. ThreadX的设计很棒，代码写得非常好，且非常规范。如所有ThreadX相关的函数都是`tx_`开头，更加符合我的代码审美
3. 官方提供完善的**中文文档**
4. 提供了从文件系统、USB到网络协议栈的全家桶，并且全都可以使用CubeMX生成和配置

当然，其他RTOS如FreeRTOS也是非常好的选择，选自己喜欢的就行，一通百通。

我们主要会用到ThreadX、FileX、LevelX、USBX等，下面我们会从Azure RTOS的核心，ThreadX，开始讲起。

## ThreadX
中文文档：https://learn.microsoft.com/zh-cn/azure/rtos/threadx

下面我们会学习整个ThreadX的中文文档。文档主要有6个章节，分别是：

第 1 章 - 简要概述 Azure RTOS ThreadX 及其与实时嵌入式开发的关系

第 2 章 - 介绍在应用程序中安装和使用 Azure RTOS ThreadX 的基本步骤（开箱即用）

第 3 章 - 详细介绍 Azure RTOS ThreadX（高性能实时内核）的功能操作

第 4 章 - 详细介绍如何将应用程序的接口应用到 Azure RTOS ThreadX

第 5 章 - 介绍如何编写 Azure RTOS ThreadX 应用程序的 I/O 驱动程序

第 6 章 - 介绍随每个 Azure RTOS ThreadX 处理器支持包一起提供的演示应用程序

### ThreadX简介
首先了解一些基础概念。

首先是实时软件的概念。实时软件实际上就是需要实时地和外部进行交互的软件，大多数嵌入式软件都属于实时软件。在出现RTOS之前，大多数实时软件都是使用C main函数内部的主循环来分配各个任务的处理时间（现在在一些简单的程序中仍然是这么做的）。这么做的问题在于由于每个事件的响应时间不一，对于大型或者复杂的程序，其时序特性就会发生变化，整个程序就会变得不稳定、难以维护。

而RTOS的引入可以解决这个问题，基于优先级的响应方式可以让重要的外部事件处理变得确定和快速。当然，现代操作系统基于进程和线程的调度方式是实时操作系统的进一步延伸，不过对于嵌入式系统，一般来说RTOS已经足够用了。

第二是线程的概念。由于在ThreadX中，并没有复杂的线程、进程之分，因此为了避免混淆，ThreadX统一使用“线程”来描述一项程序任务。

### 安装ThreadX

有两种方式：

1. 直接克隆源码
2. CubeMX

我们主要是使用CubeMX的方式。参见下面的文章：https://blog.csdn.net/wallace89/article/details/114941859。大多数的配置保持默认即可，唯一需要注意的是，`TX_TIMER_TICKS_PER_SECOND`需要改为1000，即系统Tick的时间为**1ms**，这也是大多数RTOS的默认设置。然后Memory Pool大小，即堆栈大小设置大一些就可以了。

在设置了FileX和USBX之后，生成的代码会把CubeMX自带的FatFS和USB生成的代码覆盖掉，说明在ThreadX系统中，会由FileX和USBX替代对应的模块。注意可能需要重新移植一下对应的接口。

### 第一个ThreadX程序

在使用CubeMX生成了ThreadX代码之后，我们首先看一下`main.c`:

![image-20221111212205232](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221111212205232.png)

可以看到CubeMX在主循环之前增加了一个`MX_ThreadX_Init()`，并且增加了一句提示，说`MX_ThreadX_Init()`后面的代码永远不会执行。这就说明从`MX_ThreadX_Init()`这个函数，系统就进入了ThreadX的实时操作系统中。点进`MX_ThreadX_Init()`看看代码：

```c
  /**
  * @brief  MX_ThreadX_Init
  * @param  None
  * @retval None
  */
void MX_ThreadX_Init(void)
{
  /* USER CODE BEGIN  Before_Kernel_Start */

  /* USER CODE END  Before_Kernel_Start */

  tx_kernel_enter();

  /* USER CODE BEGIN  Kernel_Start_Error */

  /* USER CODE
```

可以看到实际上，只调用了一个ThreadX的函数`tx_kernel_enter()`，这就是ThreadX的入口。这个函数相当于是裸机程序的主循环，它永远不会返回。

OK，我们现在知道了ThreadX的入口在哪里。那么下一步就是初始化系统资源。根据官方文档，我们可以使用`tx_application_define`来初始化你的第一个线程。搜索`tx_application_define`，可以发现这个函数被生成在了`AZURE_RTOS\App\app_azure_rtos.c`文件下。这个文件非常长，简单来说就是根据在CubeMX里面设置的Azure Application的设置，来初始化对应的资源：

![image-20221111213535995](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221111213535995.png)

然后在资源创建完成之后，你就可以在对应的区域添加你自己的实时任务代码了。如下图框起来的就是ThreadX核心的线程代码和FileX的线程代码。

![image-20221111213650138](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221111213650138.png)

​	一般可以使用`tx_thread_create`来创建。下面这段代码就是官方文档里面的最最简单的单线程创建代码：

```c
#include "tx_api.h"
unsigned long my_thread_counter = 0;
TX_THREAD my_thread;
main( )
{
    /* Enter the ThreadX kernel. */
    tx_kernel_enter( );
}
void tx_application_define(void *first_unused_memory)
{
    /* Create my_thread! */
    tx_thread_create(&my_thread, "My Thread",
    my_thread_entry, 0x1234, first_unused_memory, 1024,
    3, 3, TX_NO_TIME_SLICE, TX_AUTO_START);
}
void my_thread_entry(ULONG thread_input)
{
    /* Enter into a forever loop. */
    while(1)
    {
        /* Increment thread counter. */
        my_thread_counter++;
        /* Sleep for 1 tick. */
        tx_thread_sleep(1);
    }
}
```

可以看到整个的框架和CubeMX生成的实际上是一样的，入口就是`main` -> `tx_kernel_enter`，然后初始化就是`tx_application_define`，然后里面定义了一个线程`my_thread_entry`。这个线程会增加计数器 -> 休眠1Tick，就这样一直无限循环下去。

当然，你也可以创建一个线程，然后执行完毕之后就退出，这也是可以的。

### 基础配置

ThreadX中有很多配置项，这些配置项都定义在`Core/Inc/tx_user.h`中，只有当定义了`TX_INCLUDE_USER_DEFINE_FILE`后这个文件里面的设置才会生效。不过不用担心，CubeMX已经帮我们设置好了。具体都有哪些设置，[这里](https://learn.microsoft.com/zh-cn/azure/rtos/threadx/chapter2#detailed-configuration-options)看文档吧，也没什么捷径，唯一需要注意的是可以在CubeMX里面定义的尽量在CubeMX里面定义，否则重新生成代码可能会把原先的设置覆盖掉。

### ThreadX的内核组件

ThreadX应用程序包含四种类型的程序执行：

- 初始化
- 线程执行
- 中断服务程序（Interrupt Service Routine，ISR）
- 应用程序计时器

这四类程序的执行如下图：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221111221713360.png" alt="image-20221111221713360" style="zoom:66%;" />

下面简单介绍一下这四类程序。

- 初始化

  就是初始化

- 线程执行

  在初始化完毕之后，ThreadX就会进入**线程计划循环**，简单来说就是

  1. 查找ready的最高优先级的线程
  2. 把控制权转交给该线程
  3. 执行完毕（或者更高优先级的线程Ready）
  4. 执行权交回线程计划循环
  5. 继续查找下一个最高优先级的线程

  这样一个无限循环的过程。

- 中断服务程序（ISR）

  **中断是实时系统的基础**。在检测到更高优先级的中断时，系统会把当前程序的执行信息保存在堆栈上，然后去执行中断服务程序。

- 应用程序计时器

  和中断服务程序类似，区别在于其硬件实现是对程序隐藏的，通常被用来执行**超时、定时任务或者监视器服务**。此外，应用程序计时器无法相互中断，这一点也和ISR不同。

这就是ThreadX中定义的四类主要程序。每一类程序的详细讲解，可以参考[这里](https://learn.microsoft.com/zh-cn/azure/rtos/threadx/chapter3#initialization-1)的文档。

### ThreadX的设备驱动程序



## USBX

USBX是Azure RTOS提供的高性能USB主机和设备的嵌入式堆栈，其特点是

1. 内存占用小
2. 和ThreadX、FileX等Azure RTOS体系完美适配和集成
3. 支持绝大多数MCU，以及绝大多数USB设备和主机类别
4. 提供直观且一致的API，遵循动词 - 名词的命名约定，所有API均带有`ux_`前缀
5. 所有阻塞API都带有可选的超时
6. 通过了各种认证

下面就介绍一下USBX的功能，以及使用

### USBX功能

USBX 同时支持主机端和设备端。 每一端都由三个层组成。

- 控制器层
- 堆栈层
- 类层

如下图所示：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221112215822138.png" alt="image-20221112215822138" style="zoom:50%;" />

### USBX安装

Cube已经把USBX所需要的文件生成在了`USBX`目录下，在代码中只需要按需引入`ux_api.h`或者`ux_port.h`即可。

此外，要想在项目中使用USBX还需要如下步骤：

1. 在`tx_application_define`的开头添加`ux_system_initialize`，完成USB资源的初始化
   - `ux_system_initialize`这个函数定义了USBX的内存池，具体使用可以看后续章节
2. 添加对`ux_host_stack_initialize`的调用
3. 初始化所需的 USBX 类（主机和/或设备类）
4. 初始化系统中可用的设备控制器
5. （optional）修改 tx_low_level_initialize.c 文件，以添加低级别硬件初始化和中断向量路由地址

### USBX的配置

和ThreadX类似，USBX的所有配置都在`ux_user.h`文件中。可以按照[这里](https://learn.microsoft.com/zh-cn/azure/rtos/usbx/usbx-device-stack-2#configuration-options)的文档修改相应配置。修改之前注意检查CubeMX。

### USBX的初始化

#### 初始化 & 反初始化

首先是USBX的初始化。由于USBX有自己的内存管理器，因此可以为USBX单独申请内存池，这个步骤必须在**初始化主机或者设备端之前**完成。下面是初始化内存池的例子：

```c
// USBX 内存资源初始化为128K的普通内存，无缓存安全内存池
ux_system_initialize(memory_pointer,(128*1024),UX_NULL,0);
```

注意：当常规内存不是缓存安全的，且MCU需要使用DMA时，需要定义缓存安全内存池。在不定义缓存安全内存池时，USBX会使用普通内存池代替。

要想终止使用USBX（反初始化），则需要首先保证所有的USB类和控制器资源已经终止，然后执行`ux_system_uninitialize();`即可。

### USBX的设备堆栈（Device Stack）

对于我们来说，主要是使用设备堆栈，即让我们的MCU成为一个USB设备。USBX的设备堆栈的结构可以参考下图：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221114194946535.png" alt="image-20221114194946535" style="zoom:50%;" />

简单理解，可以把整个USBX的设备堆栈划分为4个部分（3层，上面已经介绍过）：

- 最上层是设备类（Device Classes）
- 中间层是设备堆栈（Device Stack）
- 底层是设备控制器（Device Controller）
- 中间还有一个是VBUS控制器

下面会先讲一下初始化和接口调用，然后简单介绍一下

#### 初始化设备堆栈

要想使用USBX的设备堆栈，需要在调用`ux_system_initialize`初始化完毕内存池之后，调用`ux_device_stack_initialize`来初始化设备堆栈所使用的所有资源。然后，就可以使用`ux_device_stack_class_register`去注册对应是**USB设备类**。还需要注意的是，USB设备控制器（即Device Controller），需要单独初始化。

说了这么多，下面给一个初始化的checklist：

- USBX内存池：`ux_system_initialize`
- USB设备堆栈：`ux_device_stack_initialize`
- 注册USB设备类到设备堆栈：`ux_device_stack_class_register`
- USB设备控制器：`ux_dcd_controller_initialize`(这个函数似乎已depreciated，查找`ux_dcd_*`获取最新更新)

下面是一个Demo：

```c
// USBX内存池初始化
ux_system_initialize(memory_pointer,(128*1024), 0, 0);

// 设备堆栈初始化
status = ux_device_stack_initialize(&device_framework_high_speed,
    DEVICE_FRAMEWORK_LENGTH_HIGH_SPEED, &device_framework_full_speed,
    DEVICE_FRAMEWORK_LENGTH_FULL_SPEED, &string_framework,
    STRING_FRAMEWORK_LENGTH, &language_id_framework,
    LANGUAGE_ID_FRAMEWORK_LENGTH, UX_NULL);

// 存储设备类的初始化参数
/* Store the number of LUN in this device storage instance: single LUN. */
storage_parameter.ux_slave_class_storage_parameter_number_lun = 1;
/* Initialize the storage class parameters for reading/writing to the Flash Disk. */
storage_parameter.ux_slave_class_storage_parameter_lun[0].ux_slave_class_storage_media_last_lba = 0x1e6bfe;
storage_parameter.ux_slave_class_storage_parameter_lun[0].ux_slave_class_storage_media_block_length = 512;
storage_parameter.ux_slave_class_storage_parameter_lun[0].ux_slave_class_storage_media_type = 0;
storage_parameter.ux_slave_class_storage_parameter_lun[0].ux_slave_class_storage_media_removable_flag = 0x80;
storage_parameter.ux_slave_class_storage_parameter_lun[0].ux_slave_class_storage_media_read = tx_demo_thread_flash_media_read;
storage_parameter.ux_slave_class_storage_parameter_lun[0].ux_slave_class_storage_media_write = tx_demo_thread_flash_media_write;
storage_parameter.ux_slave_class_storage_parameter_lun[0].ux_slave_class_storage_media_status = tx_demo_thread_flash_media_status;
// 注册上面的存储设备类到interface 0
status = ux_device_stack_class_register(ux_system_slave_class_storage_name ux_device_class_storage_entry,
    ux_device_class_storage_thread,0, (VOID *)&storage_parameter);

// 初始化设备控制器
status = ux_dcd_controller_initialize(0x7BB00000, 0, 0xB7A00000);
```

#### 应用层的接口调用

USBX的接口分为两层：

- USB Device **Stack** API：负责注册 USBX 设备组件（例如类和设备框架）。
- USB Device **Class** API：这些API随着具体的USB设备类的不同而不同，比如MSC/HID等。

大多数情况下，只需要调用Class API即可，**并不需要**调用Stack API。

#### USB的设备框架

设备框架主要是用来定义USB设备，分成4个部分：

- 设备描述符：
- 配置描述符
- 接口描述符
- 终结点描述符

这些都遵循USB协议。定义方式是这样的，下面是一个例子：

```c
#define DEVICE_FRAMEWORK_LENGTH_HIGH_SPEED 60
UCHAR device_framework_high_speed[] = {
    /* Device descriptor */
    0x12, 0x01, 0x00, 0x02, 0x00, 0x00, 0x00, 0x40, 0x0a, 0x07, 0x25, 0x40, 0x01, 0x00, 0x01, 0x02, 0x03, 0x01,
    /* Device qualifier descriptor */
    0x0a, 0x06, 0x00, 0x02, 0x00, 0x00, 0x00, 0x40, 0x01, 0x00,
    /* Configuration descriptor */
    0x09, 0x02, 0x20, 0x00, 0x01, 0x01, 0x00, 0xc0, 0x32,
    /* Interface descriptor */
    0x09, 0x04, 0x00, 0x00, 0x02, 0x08, 0x06, 0x50, 0x00,
    /* Endpoint descriptor (Bulk Out) */
    0x07, 0x05, 0x01, 0x02, 0x00, 0x02, 0x00,
    /* Endpoint descriptor (Bulk In) */
    0x07, 0x05, 0x82, 0x02, 0x00, 0x02, 0x00
};
```

## FileX

Azure FileX是一个FAT格式的文件管理系统，支持各种设备、各种FAT格式和exFAT拓展格式，并且针对性地做了很多优化。

### FileX的安装

和上面ThreadX、USBX的安装一样，要安装FileX只需要在CubeMX中开启对应的功能，并设置相关参数即可。FileX的相关文件基本都是`fx_`为开头，和上面ThreadX/USBX类似，主要引入的头文件是`fx_api.h`和`fx_port.h`。

需要额外注意的是，FileX依赖ThreadX的计时器（timer）和信号量（semaphores）功能。

要想在项目中使用FileX，还需要如下配置：

1. include `fx_api.h`文件
2. 在`tx_application_define` 或**应用程序线程**中调用 `fx_system_initialize`，来初始化FileX文件系统
3. 然后使用 `fx_media_open` 来添加文件系统FileX媒体，这个函数必须在**应用程序线程**中调用
4. 编译代码

### FileX的配置项

配置项在[这里](https://learn.microsoft.com/zh-cn/azure/rtos/filex/chapter2#configuration-options)。注意使用CubeMX配置。整个Azure RTOS的配置都是类似的，参考上面其他模块的配置即可。

### FileX的功能组件

#### FileX的文件系统 & FAT文件系统基础

FileX会把物理的存储看作是一个**逻辑扇区的数组**，至于扇区和底层物理的映射需要在I/O驱动程序中来做，这些驱动程序在调用`fx_media_open`的时候被初始化。在FileX中，底层物理存储被称为**介质（media）**。

FAT12/16/32的介质的逻辑扇区布局一般长这样：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221115224431914.png" alt="image-20221115224431914" style="zoom:50%;" />

可以看到：

1. 逻辑扇区的起始ID为1，扇区1指向的是**保留扇区（reserved sector）**
2. 第0扇区为启动扇区
   - 不过，当介质有**隐藏扇区**的时候，这些隐藏扇区会在启动扇区之前，因此启动扇区的偏移量必须要考虑是否有隐藏扇区

作为区分，exFAT的逻辑扇区长这样：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221115224953857.png" alt="image-20221115224953857" style="zoom:50%;" />

在exFAT中， 启动块和 FAT 区域属于**系统区域**。 其余群集属于**用户区域**。同时，作为对FAT12/16/32的兼容，exFAT中的0x0B 和 0x40 之间的区域（其中包含 FAT12/16/32 中的各种介质参数）在 exFAT 中标记为“保留”，**必须全都置为0**。

FAT12/16/32以及exFAT的启动记录，可以参考文档中的详细说明：https://learn.microsoft.com/zh-cn/azure/rtos/filex/chapter3，这里不再一一介绍。

### FileX 的 I/O 驱动程序

FileX支持多介质（media），每一个FileX的介质实例都有其对应的唯一的I/O的驱动程序。在FileX中，`FX_MEDIA`结构定义了管理对应的存储介质所需要的一切。用户需要做的，就是为FileX的I/O驱动程序，写一个函数作为整个驱动的入口：

```c
void my_driver_entry(FX_MEDIA *media_ptr);
```

这个函数的唯一一个参数是`FX_MEDIA`，它会根据`FX_MEDIA`中的请求`fx_media_driver_request`和具体的参数，来判断当前要做的事情，然后调用对应的函数/硬件接口。在这里可以看到常用的`fx_media_driver_request`的值，以及对应请求会用到的参数：https://learn.microsoft.com/zh-cn/azure/rtos/filex/chapter5#io-driver-requests。

`my_driver_entry`可以理解成是对硬件存储驱动程序的一个统一入口，在这里去分发不同的硬件I/O请求。需要注意的是，在CubeMX生成的LevelX + FileX代码中，入口函数名为：`fx_stm32_levelx_nor_driver`

[这里](https://learn.microsoft.com/zh-cn/azure/rtos/filex/chapter5#sample-ram-driver)是一个针对RAM的Driver `fx_ram_driver.c`，作为参考。

### FileX的容错模块

FileX包含了一个容错模块，用于在文件写入过程中介质断电或者弹出时的容错。这个模块不是默认开启的，如果想要开启，需要在定义了`FX_ENABLE_FAULT_TOLERANT`的情况下，生成FileX。这样，在调用`fx_media_open`初始化FileX之后，FileX会自动调用`fx_fault_tolerant_enable`从而启动容错服务。

容错模块会开启容错日志，容错日志会在介质中占用一个cluster的大小。容错日志里面会记录文件操作，当操作被异常终止的时候，日志就可以把当前操作记录下来，后面再初始化FileX之后就会根据容错日志来完成上次中断的操作。当所有数据都成功写入并且提交到介质之后，FileX会删除当前的日志条目文件更新操作完成。

## LevelX

LevelX是Azure RTOS提供的用于NAND和NOR闪存的磨损均衡工具。需要注意的是，LevelX不依赖FileX，但是会依赖ThreadX。

### LevelX的安装

和上面基本一样，不再赘述。生成的文件中，api都在`lx_api.h`中。此外，LevelX生成的文件可以分成两类，`lx_nand_xxxxx`和`lx_nor_xxxxx`，分别用于NAND Flash和NOR Flash的磨损均衡。按需取用即可。

在官方库里面，还有LevelX结合FileX的示例，在这些文件中

```
# nand flash
demo_filex_nand_flash.c
fx_nand_flash_simulated_driver.c
lx_nand_flash_simulator.c

# nor flash
demo_filex_nor_flash.c
fx_nor_flash_simulated_driver.c
lx_nor_flash_simulator.c
```

### LevelX的配置选项

同上，见：https://learn.microsoft.com/zh-cn/azure/rtos/levelx/chapter2#configuration-options

### LevelX对NOR Flash的支持

我们主要会使用NOR Flash，因此主要看一下LevelX对NOR Flash的支持就好。LevelX在初始化的时候，会调用`lx_nor_flash_open`，这个函数会把NOR Flash的驱动程序指定给LevelX。NOR Flash的驱动函数如下：

```c
INT nor_driver_initialize(LX_NOR_FLASH *instance);
```

其中`instance`就是LevelX的实例。在这个驱动函数里面，会把具体的硬件驱动和LevelX实例关联起来。每个LevelX的NOR Flash的实例需要实现如下几个服务：

- 读取扇区
- 写入扇区
- 块擦除
- 验证擦除的块
- 系统错误处理程序

上面的这些服务是通过在上面的初始化函数中设置`LX_NOR_FLASH`的`instance`的**相应函数指针来设置**的，此外，驱动程序的初始化函数还负责

1. 指定Flash的基础地址
2. 指定块的总数和每个块的字节数
3. RAM缓冲区设置

下面介绍一下需要在驱动程序中实现的服务。

### 读取扇区

顾名思义，就是读取NOR Flash中的特定扇区。如果成功，返回`LX_SUCCESS`。失败则返回`LX_ERROR`。其原型函数为：

```c
INT nor_driver_read_sector(
    ULONG *flash_address,
    ULONG *destination, 
    ULONG words);
```

其中，`flash_address`就是读取的逻辑扇区的地址，`destination`是读出来的数据位置，而`words`则是读取大小（32位字节的数量）。

### 写入扇区

和读取类似，如果成功，返回`LX_SUCCESS`。失败则返回`LX_ERROR`。原型函数为：

```c
INT nor_driver_write_sector(
    ULONG *flash_address,
    ULONG *source, 
    ULONG words);
```

`flash_address`为写入逻辑扇区的地址，`source`为数据，`words`为写入的大小。

> 注意，在写入时，需要自行判断写入是否成功。这个步骤可以通过读取回写入值并校验实现。

### 块擦除

负责擦除指定的块（Block）。如果成功，返回`LX_SUCCESS`。失败则返回`LX_ERROR`。原型函数为：

```c
INT nor_driver_block_erase(ULONG block,  
    ULONG erase_count);
```

`block`就是要擦除的块，而`erase_count`则主要用于诊断

### 验证擦除的块

LevelX需要验证对应的块是否已经成功擦除，如果已被擦除，则返回`LX_SUCCESS`，否则返回`LX_ERROR`。函数原型如下：

```c
INT nor_driver_block_erased_verify(ULONG block);
```

`block`就是要验证的块。

### 驱动程序系统错误

这个函数负责处理驱动程序发生的错误，函数原型入下：

```c
INT nor_driver_system_error(UINT error_code);
```

`error_code`表示发生的错误码。

### NOR的模拟驱动程序

LevelX提供了使用RAM来模拟NOR闪存的驱动程序，默认情况下，这个驱动程序会提供8个闪存块（Block），每个块有16个扇区（Sector），每个扇区512字节。这个模拟驱动程序在`lx_nor_flash_simulator.c`中定义，可以作为自定义驱动程序的很好的参考。其初始化函数为`lx_nor_flash_simulator_initialize`。

### FileX集成

` fx_nor_flash_simulated_driver.c`文件提供了NOR模拟 + FileX的集成驱动程序，可以作为很好的参考。

另外，官方问题还有如下提示，可以用来提升读写效率：

>FileX NOR 闪存格式应该是小于 NOR 闪存提供的扇区的一个完整块大小。 这将有助于确保在损耗均衡处理期间获得最佳性能。 在 LevelX 损耗均衡算法中提高写入性能的其他技巧包含：
>
>1. 确保所有写入大小都恰好是一个或多个群集（cluster），并且在确切的群集边界上启动。
>2. 通过  FileX 的API `fx_file_allocate` 类执行大文件写入操作之前，请先预分配群集。
>3. 定期使用 `lx_nor_flash_defragment` 释放尽可能多的 NOR 块，从而提高写入性能。
>4. 确保已启用 FileX 驱动程序以接收释放扇区信息，并通过调用 `lx_nor_flash_sector_release` 在驱动程序中处理对驱动程序发出的释放扇区的请求。

## CubeMX的坑

在使用CubeMX生成ThreadX工程的时候，需要注意如下几个坑：

1. 生成的ThreadX，默认的porting是IAR，即使在CubeMX中选择makefile，也一样。因此，需要去官方库，把`port/cortex_m7/gnu`目录下的内容复制到CubeMX工程下的`Middlewares/ST/threadx/ports/cortex_m7/gnu`。

2. 把`makefile`里面的IAR相关的路径干掉，换成gnu的

3. 在`makefile`里面添加`.S`文件（即ThreadX的gnu porting）相关的编译选项，代码如下：

   ```makefile
   # 添加所有的 .S 文件到 S_SOURCES，主要是 gnu porting。
   # 别忘记 tx_initialize_low_level.S 这个文件。
   S_SOURCES = $(wildcard Middlewares/ST/threadx/ports/cortex_m7/gnu/src/*.S) \
   Core/Src/tx_initialize_low_level.S
   
   # .. 中间省略，下面到生成的 OBJECTS 的地方，添加 .S 文件对应的 OBJECTS
   OBJECTS += $(addprefix $(BUILD_DIR)/,$(notdir $(sort $(S_SOURCES:.S=.o))))
   vpath %.S $(sort $(dir $(sort $(S_SOURCES))))
   
   # .. 然后到构建命令这边，添加 .S 对应的构建任务（参照 .s 即可）
   $(BUILD_DIR)/%.o: %.S Makefile | $(BUILD_DIR)
   	$(AS) -c $(CFLAGS) $< -o $@
   ```