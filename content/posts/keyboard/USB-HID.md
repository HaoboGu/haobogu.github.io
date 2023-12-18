---
title: "USB-HID 设备"
author: "Haobo Gu"
tags: [keyboard, embedded]
date: 2022-12-20T21:03:23+08:00
summary: USB HID 设备开发
draft: true
---

# USB HID 设备开发

## USB基础概念

### USB的传输（Transfer）、事务（Transaction）、包（Pack）

USB通信的数据交互的三层概念。从大往小依次是传输 -> 事务 -> 包，即一次传输可以有多个事务，而每个事务可以包含多个USB包。

### USB 主机（HOST）和设备（Device）

一个USB连接是由主机和设备构成。其中，主机是核心，所有通信必须由主机主动发起。只有主机询问了，设备才可以向主机发送数据。

在设备中，存储有**设备描述符**来描述每个设备的样子，每个设备只能有一个设备描述符。

### 配置（Configuration）

配置是用来描述当前USB设备（Device）的作用。同一个设备可以有多个配置，即可以完成不同的功能，但是同一时刻只能有一个配置生效。

需要注意的是，USB设备当前使用哪个配置，是由USB主机来确定的。如果把USB主机当做领导，USB设备当做小弟，那么就可以理解成领导可以让小弟写代码（配置1）、画图（配置2），但是小弟同一时刻只能做一件事情。

一般来说，一个USB设备只有一个配置，而多功能的实现需要使用下面介绍的接口（Interface）。

### 接口（Interface）

接口是USB设备完成的具体的功能，它在配置之下，即一个配置可以有多个接口。比如写代码，小弟可以写C、也可以写Python。这就是写代码（同一个配置）下的多个接口（不同语言）。

### 端点（Endpoint）

USB设备的一个接口完成一种功能，每个接口必须配置**一个或多个端点**，用于USB主机和USB设备的通信。端点可以理解成是USB设备某个接口（功能）的缓冲区，它是绑定在接口上的。在USB协议中，有**端点描述符**来描述这段缓冲区的属性。

### USB的各种描述符（Descriptor）

USB的描述符实际上就是用来描述USB相关的各种配置、接口、端点、选项等，从而让USB主机和设备能够正常建立起通讯。USB描述符非常复杂，包含很多内容，我们只需要了解基础的即可。对于我们需要学习的HID设备，除了基础描述符之外，还包含了HID描述符、报告描述符、物理描述符。而这三个描述符就是USB HID设备的核心，是需要重点学习的。下面就是一个USB HID设备的描述符图示：供参考：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221221200257918.png" alt="image-20221221200257918" style="zoom:33%;" />

可以看到，有设备描述符、配置描述符、接口描述符、端点描述符、HID描述符、报告描述符、物理描述符、以及额外的字符串描述符（这个描述符就是一些字符串而已）

这些描述符后面再讲，现在先知道名字。


## USB描述符

USB描述符实际上就是一些C语言里面的结构体或者数组，其内容就是描述当前的设备有哪些特征。下面就讲一下USB HID设备会遇到的各种描述符。

### 设备描述符

设备描述符是USB主机枚举USB设备所申请的第一个描述符，这个描述符长度为18个字节，定义了USB设备以及通讯的基础信息。详细内容如下：

![image-20221221200939937](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221221200939937.png)

在STM32的代码中，如果是CubeMX生成的代码，搜索`USBD_HS_DeviceDesc`，如果是USBX，搜索`USBD_DeviceDesc`。需要注意的是，USB描述符里面都是小端模式，即先低后高。见上面`idVendor`、`idProduct`的定义。下面CubeMX示例

```c
// STM32CubeMX生成的USB设备描述符部分的代码
__ALIGN_BEGIN uint8_t USBD_HS_DeviceDesc[USB_LEN_DEV_DESC] __ALIGN_END =
{
  0x12,                       /*bLength */
  USB_DESC_TYPE_DEVICE,       /*bDescriptorType*/
  0x00,                       /*bcdUSB */
  0x02,
  0x00,                       /*bDeviceClass*/
  0x00,                       /*bDeviceSubClass*/
  0x00,                       /*bDeviceProtocol*/
  USB_MAX_EP0_SIZE,           /*bMaxPacketSize*/
  LOBYTE(USBD_VID),           /*idVendor*/
  HIBYTE(USBD_VID),           /*idVendor*/
  LOBYTE(USBD_PID_HS),        /*idProduct*/
  HIBYTE(USBD_PID_HS),        /*idProduct*/
  0x00,                       /*bcdDevice rel. 2.00*/
  0x02,
  USBD_IDX_MFC_STR,           /*Index of manufacturer  string*/
  USBD_IDX_PRODUCT_STR,       /*Index of product string*/
  USBD_IDX_SERIAL_STR,        /*Index of serial number string*/
  USBD_MAX_NUM_CONFIGURATION  /*bNumConfigurations*/
};
```

大概对设备描述符整体有了了解之后，下面详细深入每个字段。

#### bLength

设备描述符长度，设备描述符是18字节，即十六进制`0x12`

#### bDescriptorType

本描述符的类型。设备描述符的话就是`0xa01`。所有的描述符类型见下表：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221221201902457.png" alt="image-20221221201902457" style="zoom:50%;" />

#### bcdUSB

USB的协议版本，表示形式为0xJJMN，JJ为主要版本号，M为次要版本号，N为次要版本。如USB2.0就是`0x0200`，USB1.1是`0x0110`，USB3.11是`0x0311`

#### bDeviceClass、bDeviceSubClass、bDeviceProtocol

这三个字段分别是USB设备的类别、子类和协议，他们联合起来定义了USB设备要做的事情。

其中：

- USB的类别包含如人机交互类、图像类、音频类等；

- USB子类是类别下的子类，如音频类下面的音频控制、音频流等
- USB设备协议，如人机接口类中的鼠标、键盘、触摸屏等，都是不同的协议

USB设备类信息可以参考：https://www.usb.org/defined-class-codes

对于HID设备来说，由于有一些需要可以在Boot时能用，因此有一些额外配置。可以参考：https://www.usb.org/document-library/device-class-definition-hid-111 。简单来说就是，如果想要在Boot中能用，那么bDeviceSubClass为`0x01`，bDeviceProtocol为`0x01`（键盘）或`0x02`（鼠标）。

bDeviceClass如果设置为0，那么这三个的定义就使用**接口描述符**里面的定义。

#### bMaxPackeSize0

端点一次最大能传输多少字节，端点0最低为8字节。这个参数和USB本身的**速度等级**（低速、全速、高速）以及**传输类型**（控制传输、数据传输等）有关。

唯一需要注意的是，**控制传输**一般使用端点0。其他配置见下表：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221221203518128.png" alt="image-20221221203518128" style="zoom:50%;" />

#### idVender、idProduct、bcdDevice

厂商ID、产品ID以及产品版本号，除了厂商ID需要交费申请外（当然也可以随便写），其他自由定义。如果使用STM32，VID可以使用ST的。

#### iManufacturer、iProduct、iSerialNumber

描述厂商、产品的字符串的索引以及序列号字符串的索引，没有用0即可。

#### bNumConfiguration

这个USB设备的配置的数量，一般为1

### 配置描述符

一个USB设备至少有一个配置，配置的数量由设备描述符里面的bNumConfiguration确定。每个配置都对应一个**配置描述符集合**。为什么叫配置描述符**集合**呢？这是因为一个USB设备的配置，实际上包含了标准配置描述符、接口描述符、端点描述符，对于HID设备还包含HID描述符。设备描述符集合由USB主机直接请求，然后**<u>USB设备会直接把整个配置描述符集合返回</u>**，不会单独返回配置描述符、接口描述符等。

这里，我们先只介绍配置描述符。单个配置描述符有9个字节，其详细定义如下：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221222200910171.png" alt="image-20221222200910171" style="zoom:50%;" />

下面详细介绍一下。

#### bLength

描述符长度，和设备描述符一样

#### bDescriptorType

描述符类型，可以参考上面设备描述符这个章节的介绍。对于配置描述符来说，这个字段为`0x02`。

#### wTotalLength

配置描述符集合总长度。上面已经提到了，配置描述符集合包含配置描述符、接口描述符、端点描述符，对于HID设备还包含HID描述符等等，因此这里的总长度就是上面这一堆描述符的总长度。

#### bNumInterfaces

当前配置下的接口数目。一般来说，一个单一功能的设备只会有一个接口，如键盘、鼠标等都只对应一个接口。而复合设备可以有多个接口，比如键盘+鼠标的复合设备。

在实现的时候，需要注意这一点。单个功能的设备都写到一个接口中。

#### bConfigurationValue

这个是当前配置的标识符。在一个USB设备有多个配置的时候，这个Value就作为配置的标识。

#### iConfiguration

描述当前配置的字符串索引，如果没有，则为0

#### bmAttributes

配置的一些特性，其中D7、D0~D4为保留位，D7为1，D0~D4为0。

D5：是否支持远程唤醒，1为支持

D6：供电方式，0为自供电，1为总线供电

#### bMaxPower

这个配置下，设备需要的电流，单位是2ma。一个字节最多表示255，就是大约500ma。500ma也是USB默认情况下的最大电流，实际上就是在这里定义的。

### 接口描述符

接口描述符在配置描述符集合中，不能被单独请求，只能随着配置描述符集合一起返回给USB主机。它也是有9个字节，定义如下：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221222202748960.png" alt="image-20221222202748960" style="zoom:50%;" />

#### bLength、bDescriptorType

和上面相同

#### bInterfaceNumber、bAlternateSetting

bInterfaceNumber为当前接口的编号，如果一个配置有多个接口的话，那么每个接口都是一个独立的编号。编号从0开始。

bAlternateSetting为备用编号，一般为0。

#### bNumEndpoints

该接口使用的端点的个数。接口需要分配端点来实现具体的功能。

需要注意的是，这里的个数不包括端点0。即如果这个值为1，那么实际上存在端点0和端点1两个端点（端点0一般是控制端点），这个后面再讲。

#### bDeviceClass、bDeviceSubClass、bDeviceProtocol

当设备描述符的bDeviceClass为0时，这三个字段就被用来标识USB设备的类别。具体用法参见上面设备描述符的介绍。

#### iInterface

描述这个接口的字符串的索引，如果没有，则为0

### 端点描述符

端点描述符在配置描述符集合中，性质和接口描述符类似。它包含7个字节，定义如下：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221222203709517.png" alt="image-20221222203709517" style="zoom:50%;" />

#### bLength、bDescriptorType

同上

#### bEndpointAddress

端点的地址定义。

Bit0~3：端点编号

Bit4~6：保留，为0

Bit7：如果端点不是控制端点，则表示数据传输方向。0：OUT endpoint，1：IN endpoint；如果是控制端点，则忽略

#### bmAttributes

端点的属性。

Bit0~1：传输类型（Transfer Type）

- 00：控制传输 Control
- 01：同步传输 IsoChronous
- 10：批量传输 Bulk
- 11：中断传输 Interrupt

Bit2~7：如果是非同步端点，则为0；如果是同步端点，参考 https://www.usbzh.com/article/detail-56.html

#### wMaxPackeSize

此端点能够接收或发送的数据包大小。

这个定义很细节，和端点类型也有关系。参考 https://www.usbzh.com/article/detail-56.html

#### bInterval

USB主机轮询端点的时间间隔。对于高速设备，周期单位为125us，而对于低速、全速设备，周期单位为1ms。

USB主机在枚举设备的时候，会根据这里的设置来定时向端点请求数据。其设置的值和端点类型有关：

- 高速、全速设备，同步端点：`0x01~0x10`，其表示值为2的(bInterval-1)次方。即如果bInterval为1，则表示轮询间隔为1个周期（1ms for 全速，125us for 高速）；如果值为4，则轮询间隔为8个周期（8ms for 全速、1000us=1ms for 高速）。
- 高速设备，中断端点：同上，其表示值为2的(bInterval-1)次方
- 低速、全速设备，中断端点：`0x01~0xff`，即1~255ms轮询一次
- 高速设备，批量/控制输出端点：bInterval指定端点的最大[NAK](https://www.usbzh.com/article/detail-453.html)速率。0表示永不NAK，其他值表示 bInterval * 125us 内最多1个NAK
- 其他：无意义，随便设置

完整的各项配置含义，参见USB官方文档的介绍：https://www.usb.org/document-library/usb-20-specification （usb_20.pdf中的271页，table9-13）。

### HID描述符

USB HID设备，即人机接口设备（Human Interface Devices），是人和机器交互使用的设备，如鼠标、键盘、游戏手柄等。由于HID设备要求用户的输入能够及时地被响应，因此其传输方式通常采用**中断传输**。

默认情况下，HID设备的定义会放在**接口描述符**中，其设备描述符中的bInterfaceClass/bInterfaceSubClass/bInterfaceProtocol均为0。

在接口描述符中，上述三个字段配置如下：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221223191810077.png" alt="image-20221223191810077" style="zoom:50%;" />

对于HID设备，在配置描述符集合中还必须返回HID描述符。HID描述符的组成如下：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221223191932575.png" alt="image-20221223191932575" style="zoom:50%;" />

下面详细介绍。

#### bLength、bDescriptorType

同上

#### bcdHID

HID设备所遵循的HID版本号，为4位16进制的BCD码。1.0即0x0100，1.1即0x0101，2.0即0x0200。

#### bCountryCode

HID设备的国家/地区的代码，在[这里](https://www.usb.org/sites/default/files/documents/hid1_11.pdf)查询。大多数情况下使用0即可，即不限定地区。

#### bNumDescriptors

HID设备支持的下级描述符的数目。什么是下级描述符呢？ 对于HID描述符来说，其下级描述符指的是**报告描述符**和**物理描述符**。HID设备至少有一个报告描述符，因此bNumDescriptors至少为1。

HID设备可以有多个报告描述符和物理描述符，但一般来说，HID设备会有一个报告描述符，物理描述符很少用到。

#### bDescriptorType、wDescriptorLength

bNumDescriptors的后面就是若干个bDescriptorType和wDescriptorLength对，用来描述下级描述符的类型和长度。如果bNumDescriptors=2，那么后面就有2对bDescriptorType和wDescriptorLength。bDescriptorType的定义和上面都一样，wDescriptorLength就是对应的下级描述符的总长度。

### 报告描述符

下面介绍一下HID描述符的下级描述符之一，报告描述符。报告描述符是每个HID设备都会有的描述符，它被用来描述HID设备**报告的数据的用途及属性**。USB主机在对HID设备进行枚举的时候，会根据报告描述符来确定给HID设备发送的数据结构，以及解析后续HID设备发来的数据。

报告描述符非常复杂，下面仅做简单介绍。细节可以参考USB官网的**hid1_11.pdf和hut1_21_0.pdf**。

#### 报告描述符中的通用项（Item）

报告描述符的主要组成部分就是**通用项（item）**，可以说，报告描述符实际上就是由一个个的通用项组成的。每个通用项会描述一个或者多个**相同功能**的数据，内容包括数据的用途、属性等等。

#### 通用项的组成

通用项有两种类型，分别叫长通用项（Long Item）和短通用项（Short Item）。下面是两种通用项的组成结构：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221225202612648.png" alt="image-20221225202612648" style="zoom:50%;" />



一般来说，短通用项用的比较多。下面主要介绍短通用项。

#### 短通用项简介

在短通用项中，其第一个bytes定义了这个item的大小和类型。其中：

- Bits0~1：后面数据的大小，最多为`0x11`，即4bytes
- Bits2~3：这个通用项所属于的大类（type），有三种类型：Main、Global、Local
- Bits4~7：这个通用项的子类，也称之为标签（tag）。上面的每一个大类下面都有若干个子类，如Main大类下面会有Input、Output等各种子类。这里的类别是HID报告描述符通用项的核心，下面会详细介绍

#### 短通用项中的Main、Global、Local类型

重点：对于短通用项，一般首先用Global、Local标签来定义数据的各种属性，最后用一个Main标签来说明输入输出。Main标签的出现就代表着一个数据项的结束。

下面先简单介绍一下这三类标签

##### Main

Main主要是数据项的**描述**和**管理**。

- 描述（Input、Output、Feature）：这个数据项到底是输入还是输出
- 管理（Collection、End Collection）：把多个数据项组织到一起

##### Global

Global主要用来描述数据项的用途、数据的逻辑范围、数据的物理范围、数据的大小和个数等等。Global的用法和全局变量非常像，就是一旦定义之后，会一直生效，直到重新定义或者被Local覆盖。

##### Local

Local同样用来描述数据项，其内容和Global是一样的，区别在于Local描述的数据只在本数据项内生效，超出范围就不生效了。如果说Global定义的数据属性是全局变量，那么Local定义的数据属性就是局部变量。

#### 子类标签

上面介绍的这三个类别，是第一个bytes中的type，即大类。在他们后面，会紧跟当前通用项的标签（tag），即子类。通用项的子类有非常多，下面介绍一下比较常用的一些。这些通用项会定义具体通信时的数据项的各种属性。

##### Usage Page 和 Usage

这两个标签是用来描述数据项的具体类型和用法。这个标签比较复杂，Usage Page变化的时候，Usage也会随之变化。具体可以参考USB IF官网上 HID Usage Table 这个文档

##### Logical Maximum 和 Logical Minimum

描述数据取值的最大、最小值，这两个值就定义了当前要报告的数据的范围。比如一个设备报告电压为1V，而一个单位为2mV，则Logical Maximum为500。如果要报告一个1V ~ 1.5V的范围，则就是Logical Minimum和Logical Maximum分别为500、750。

##### Usage Maximum 和 Usage Minimum

描述数据的本身的最大、最小值。举个例子，如果要定义一个键盘，共102键。那么这两个值可以设置成0x00和0x65（即十进制101），表示一共的按键数。而上面的Logical Maximum 和 Logical Minimum则为0和1，表示按键的取值（0为未触发，1为按下已触发）。

##### Report Size

以bit为单位的指定报告数据的大小

##### Report Count

报告的同类型数据的数量

##### Report ID

报告的ID，用于区分同一个设备的不同HID功能。由于一个报告描述符可以描述多个HID功能，因此需要Report ID来对多个HID功能进行区分。

举个例子，如果一个HID设备是键鼠一体的设备，那么就可以使用Report ID来把键盘和鼠标的数据描述分开。设备在发送数据的时候，其第一个字节永远都是Report ID。

##### Collection、End Collection

用于对数据项进行组织和管理，两者需要一起使用。

#### 数据项示例

只是讲一下数据项的标签可能不够直观，下面用一个键盘的数据项例子来具体说明一下各项的功能、作用。首先看一下键盘的设备报告描述符：

```c
// 前三个通用项，定义了USB HID报告数据项的类型、子类型，然后创建了一个集合
0x05, 0x01,        // Usage Page (Generic Desktop Ctrls)。用途：通用桌面设备
0x09, 0x06,        // Usage (Keyboard)。子用途：键盘
0xA1, 0x01,        // Collection (Application)。创建了一个集合

// 集合中的第一个数据，定义了键盘的修饰键的数据。一共包含2个字节16bits，其中前8bits定义的修饰键的值，而后8bits是常量（推测是站位使用）
0x05, 0x07,        //   Usage Page (Kbrd/Keypad)
0x19, 0xE0,        //   Usage Minimum (0xE0)
0x29, 0xE7,        //   Usage Maximum (0xE7)
0x15, 0x00,        //   Logical Minimum (0)
0x25, 0x01,        //   Logical Maximum (1)
0x95, 0x08,        //   Report Count (8)。报告的个数，上面Usage一共定义了8个修饰键，那个数就是8。共计 8 * 1 = 8bits
0x75, 0x01,        //   Report Size (1)。报告的大小，由于上面逻辑值最大为1，那么1bit就够了
0x81, 0x02,        //   Input (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)。数据方向：输入。参数为括号里面的内容，注意第一项是Data，即数据
0x95, 0x01,        //   Report Count (1)。下面三行又定义了一个站位用的数据，为常量。数据大小为 8 Bits。由于报告有继承性，其他没有定义的和上面保持一致
0x75, 0x08,        //   Report Size (8)
0x81, 0x01,        //   Input (Const,Array,Abs,No Wrap,Linear,Preferred State,No Null Position)。数据方向：输入。参数为括号里面的内容，注意第一项是Const，即常量

// 集合中的第二个数据项，描述键盘的按键按下状态。一共6个字节，每个字节表示一个按键。即键盘可以同时识别6个按键（6键无冲）。每个字节上的数据值的范围在0~255。
// 这里需要注意的是，根据HID键盘的定义，如果按下的键太多，导致键盘扫描系统无法区分按键时，则全部返回0x01，即6个0x01。这样，这6个字节最多能同时表示254个键中的6个被同时按下的情况（0x02~0xff共254个按键）
// 如果有一个键按下，则这6个字节中的第一个字节为相应的键值（具体的值参看HID Usage Tables），如果两个键按下，则第1、2两个字节分别为相应的键值，以次类推。
0x05, 0x07,        //   Usage Page (Kbrd/Keypad)
0x19, 0x00,        //   Usage Minimum (0x00)
0x29, 0xFF,        //   Usage Maximum (0xFF)
0x15, 0x00,        //   Logical Minimum (0)
0x26, 0xFF, 0x00,  //   Logical Maximum (255)
0x95, 0x06,        //   Report Count (6)
0x75, 0x08,        //   Report Size (8)
0x81, 0x00,        //   Input (Data,Array,Abs,No Wrap,Linear,Preferred State,No Null Position)。数据方向：输入。参数为括号里面的内容，Array，即列表


// 集合中的第三个数据项，描述LED灯的状态。一共1个字节8bits。其中前5bits为LED状态（HID默认定义了从NumLock到Kana共5个灯），而后3bits是常量，站位使用。数据方向为输出。
0x05, 0x08,        //   Usage Page (LEDs)
0x19, 0x01,        //   Usage Minimum (Num Lock)
0x29, 0x05,        //   Usage Maximum (Kana)
0x95, 0x05,        //   Report Count (5)
0x75, 0x01,        //   Report Size (1)
0x91, 0x02,        //   Output (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)
0x95, 0x01,        //   Report Count (1)
0x75, 0x03,        //   Report Size (3)
0x91, 0x01,        //   Output (Const,Array,Abs,No Wrap,Linear,Preferred State,No Null Position,Non-volatile)

// 最后结束集合
0xC0,              // End Collection

// 64 bytes
```

根据上面的定义，键盘的每个报告数据包，实际上包含共9个字节。前8个字节表示按键状态，最后一个字节表示LED的状态。下面举几个例子说明按键的状态：

```c
// 只按下左shift的时候（根据定义，左shift在修饰键数据的第二个bit）
按下时发送一次：0x02 0x00 0x00 0x00 0x00 0x00 0x00 0x00
弹起时发送一次：0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
  
// 只按下A键时
按下时发送一次：0x00 0x00 0x04 0x00 0x00 0x00 0x00 0x00
弹起时发送一次：0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00

// 按下左shift + A键时
按下时发送一次：0x02 0x00 0x04 0x00 0x00 0x00 0x00 0x00
弹起时发送一次：0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00
```

总之，弹起是全是0x00，按下时对应的bit置1即可。

这里也有一个鼠标的例子，讲得非常清楚，可以同时参考，有助于更好地理解HID 报告描述符：https://zhuanlan.zhihu.com/p/27568561

## 总结
到现在为止，USB HID协议基本上已经介绍完了。下面复习一下要点，加强一下记忆。

1. USB描述符是USB设备用于描述自己的属性、用途、通信数据等，因此每一个USB设备必须实现其描述符，然后在USB主机枚举该设备的时候，USB主机才可以知道这个设备是什么、如何和这个设备通信。
2. 每一个USB设备只能有一个设备描述符，向主机说明设备类型、USB设备的配置的数量等。
3. 每一个USB设备至少有一个或者多个配置描述符，向主机说明当前配置下USB设备的属性、电流、接口数、配置描述符集合长度等。在同一时间只能选择一个配置生效。
4. 每一个配置下一般都会有一个配置描述符集合，配置描述符集合包括**标准配置描述符、接口描述符、端点描述符、HID描述符**。
5. 标准配置描述符是配置描述符集合的第一个描述符，标准配置描述符中有配置描述符集合的总长度，USB主机会根据这个总长度来获取配置描述符集合中的所有描述符的信息。
6. 每一个USB配置至少有一个后者多个接口描述符，向主机说明设备的类型（如果设备描述符里面没有）、此接口的端点数（端点0不算）等。每一个接口可以理解为是一个USB配置下的一种功能，如果一个USB设备有多功能，一般都是通过多个接口实现的。
7. 每一个USB接口至少有0个或者多个端点描述符，描述端点的各种属性。由于端点0默认存在，因此端点描述符可以有0个。
8. 字符串描述符用于描述设备的一些属性，包括厂商、设备名称等。
9. HID描述符只有HID设备才会存在，描述了HID设备的类型、数据格式等。
10. HID设备至少有一个报告描述符，报告描述符的主要作用就是描述HID报告，即主机和HID设备交互的数据。报告描述符使用通用项来告诉主机，在通信的数据包中，哪些位是做什么的。