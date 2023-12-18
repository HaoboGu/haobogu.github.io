---
title: "OpenOCD"
author: "Haobo Gu"
tags: [keyboard, embedding]
date: 2022-10-03T19:54:23+08:00
summary: Learn OpenOCD
---

# OpenOCD

OpenOCD是一款开源的针对嵌入式设备的调试器，可以用来烧录、调试很多嵌入式设备。

## 安装

在mac上面安装OpenOCD非常简单，使用`brew`安装就好。需要注意的是，`brew`里面默认的版本比较老旧，在安装时最好使用:

```shell
brew intall openocd --HEAD
```

在Windows上面安装则更为简单：去OpenOCD的Github的Release页面下载最新版本的OpenOCD安装文件即可。

## OpenOCD入门

OpenOCD内置了对很多MCU和开发板的支持，这些都可以去`${安装目录}/share/openocd/scripts`下获取。一般来说，只需要在`interface`文件夹下面找到你使用的debugger，然后在`board`文件夹下面找到你所使用的板子或者在`target`目录下找到你使用的MCU，即可以不做任何改动，直接使用OpenOCD进行烧录和调试：

```shell
openocd -f interface/cmsis-dap.cfg -f target/stm32h7x.cfg 
```

这是默认选项，也是大多数人的用法。不过在这里，如果我们想要对OpenOCD的烧录、调试选项做更加深入细致地定制的话，就必须熟悉OpenOCD的配置文件，也就是`-f`选项后面的文件。OpenOCD自带的这些文件是很好的参考，后面我们也会参考这些文件，针对我们自己的板子写出对应的配置文件。

## OpenOCD的配置文件

OK，到现在为止我们已经入门了OpenOCD，现在就进一步学习OpenOCD的配置文件。默认情况下（即不使用`-f`指定配置文件时），OpenOCD会使用当前目录下的`openocd.cfg`文件作为配置文件。

在写我们自己的配置文件时，我们仍然可以复用`scripts` 目录下的配置文件，像这样：

```
# 引用stlink.cfg
source [find interface/stlink.cfg]
```

大多数情况下，你所使用的debugger的配置文件都会随OpenOCD提供，引用即可。

下面，针对我们使用的debugger，需要去指定通信类型。比如，对于stlink，我们使用`hla_swd`：

```
# Choose transport interfance 
transport select hla_swd
```

或者对于`cmsis-dap`，我们使用`swd`：

```
source [find interface/cmsis-dap.cfg]
transport select swd
```

再然后，就可以看一下我们使用的MCU是否在targets里面了。绝大多数情况下，是在的，那么我们可能需要进行一些些的配置。比如，指定芯片名称：

```
set CHIPNAME stm32h7x
```

再比如，打开octo-spi的开关：

```
# Enable stmqspi
if {![info exists OCTOSPI1]} {
	set OCTOSPI1 1
	set OCTOSPI2 0
}
```

然后直接引用对应的内置配置文件即可：

```
# Use built-in stm32h7 openocd configs
source [find target/stm32h7x.cfg]
source [find board/stm32h7x_dual_qspi.cfg]
```

这里，由于我们已经事先看了这两个文件，知道打开octo-spi要把`OCTOSPI1`配置为1。对于不同的MCU，最好去看一下对应`target`下的配置文件，这样你就能够知道可以配置哪些东西，需要配置哪些东西。

OpenOCD提供了非常非常多的配置选项，其内置的各种target的选项就是非常好的参考。如果发现有命令不太明白，可以去官方文档查询。当你能够看明白一个MCU的完整的配置文件，那么相信自己写一写也不在话下了。

## OpenOCD的命令

在写完配置文件之后，我们就可以使用OpenOCD进行实际的调试了。在调试之前，我们需要学习一下最常用的OpenOCD的命令，这些命令在调试中都非常常用。

- halt：暂停CPU运行，在执行烧录命令之前必须先halt，否则CPU不会理你的

- flash：flash实际上包含很多子命令，常用的有`flash info`，`flash list`，`falsh banks`, `flash write_image`，`flash verify_image`等。具体用法参见官方手册

  - flash write_image erase [filename] [address]：烧录指定文件到对应地址。对于stm32h7，如果烧录elf文件，不需要加地址。而如果烧录bin文件，则需要加地址：

  ```
  flash write_image erase build/h7b0.bin 0x08000000
  # 或者
  flash write_image erase build/h7b0.elf
  ```

- reset：复位

- init：初始化MCU

- reset halt：复位并且立刻暂停CPU

- mdx(mdd/mdw/mdh/mdb)：显示对应地址的数据（memory display），mdd是64bit，mdw是32bit，mdh是16bit，mdb是8bit

- mwx(mwd/mww/mwh/mwb)：往对应地址写入，各个命令和上面类似

- program: OpenOCD提供了program命令，相当于是flash命令的高级封装。可以直接使用如下一条命令，完成初始化、停止、烧录、重启、退出等一系列命令可以完成的事情：

  ```shell
  # 一条命令完成烧写
  program build/h7b0.elf verify reset exit
  
  # 下面一堆命令也一样
  init
  halt
  flash write_image erase build/h7b0.elf
  reset
  shutdown
  ```

## stmqspi

OpenOCD支持stm32系列MCU的qual-spi和octo-spi，可以使用spi把外部flash映射到内部的地址空间，这些flash可以自动地被OpenOCD检测到。在这种模式下，MCU可以直接读取对应的内存，执行flash里面的代码。不过需要注意的是，**MCU不能直接从外置flash区域启动**。因此，在具体的实现代码里面，**必须在内置flash区（或者说boot代码里面），配置好qspi或者ospi的memory mapping**。这样OpenOCD才能使用地址映射去操作这部分内存。下面是测试命令：

```shell
# 首先，进入OpenOCD
nc localhost 4444

# 之后，可以使用如下命令查看spi-flash

# 查看flash列表，可以看到有内置和外置两个flash
$ flash list
> {name stm32h7b0xx.bank1.cpu0 driver stm32h7x base 134217728 size 0 bus_width 0 chip_width 0 target stm32h7b0xx.cpu0} {name stm32h7b0xx.octospi1 driver stmqspi base 2415919104 size 0 bus_width 0 chip_width 0 target stm32h7b0xx.cpu0}

# 获取内置flash的信息
$ flash info 0
> Device: STM32H7Ax/7Bx
flash size probed value 128k
STM32H7 flash has a single bank
Bank (0) size is 128 kb, base address is 0x08000000
#0 : stm32h7x at 0x08000000, size 0x00020000, buswidth 0, chipwidth 0
        #  0: 0x00000000 (0x8000 32kB) not protected
        #  1: 0x00008000 (0x8000 32kB) not protected
        #  2: 0x00010000 (0x8000 32kB) not protected
        #  3: 0x00018000 (0x8000 32kB) not protected
STM32H7Ax/7Bx - Rev: unknown (0x1001)

# 获取外部flash信息
$ flash info 1
> flash1 'win w25q64fv/jv' id = 0x1740ef size = 8192 KiB
#1 : stmqspi at 0x90000000, size 0x00800000, buswidth 0, chipwidth 0
        #  0: 0x00000000 (0x10000 64kB) not protected
        #  1: 0x00010000 (0x10000 64kB) not protected
        ... 省略
        #126: 0x007e0000 (0x10000 64kB) not protected
        #127: 0x007f0000 (0x10000 64kB) not protected
flash1 'win w25q64fv/jv', device id = 0x1740ef, flash size = 8192Ki B
(page size = 256, read = 0x03, qread = 0xeb, pprog = 0x02, mass_erase = 0xc7, sector size = 64 KiB, sector_erase = 0xd8)
```

可以看到，OpenOCD甚至连外部flash的芯片型号（w25q64）都能都读取到。除了`list`和`info`命令外，还可以使用`banks`和`probe`命令：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221003222853401.png" alt="image-20221003222853401" style="zoom:50%;" />

这里需要注意的是，在OpenOCD进行烧录的时候，通常不会启动芯片。因此对于外置SPI Flash，一般还需要通过手动写寄存器的方式来进行时钟的初始化和SPI外设的初始化。这样才能让OpenOCD正确地识别到目标芯片。如果你使用的芯片在OpenOCD已经内置，那么一般可以直接使用，直接`source`即可。如果没有内置或者你有自己的设置（比如SPI GPIO配置不同），那么就需要自己写了。下面是一个例子：

```tcl
# OCTOSPI 初始化
proc octospi_init { octo } {
	global a b
	mmw 0x58024540 0x000006FF 0				;# RCC_AHB4ENR |= GPIOAEN-GPIOKEN (enable clocks)
	mmw 0x58024534 0x00284000 0				;# RCC_AHB3ENR |= IOMNGREN, OSPI2EN, OSPI1EN (enable clocks)
	sleep 1									;# Wait for clock startup

	mww 0x5200B404 0x03010111				;# OCTOSPIM_P1CR: assign Port 1 to OCTOSPI1
	mww 0x5200B408 0x00000000				;# OCTOSPIM_P2CR: disable Port 2

	# AF mapping can be found in datasheet. For h7b0, see DS13196, page 54
	# PB02: OCSPI1_CLK, PB06: OCSPI1_NCS, PD11: OCSPI1_IO0, PD12: OCSPI1_IO1, PE2: OCSPI1_IO2, PD13: OCSPI1_IO3
	# Generate the GPIO config: perl resources/gpio_gen.pl -c "PB06:AF10:V, PB02:AF09:V, PD13:AF09:V, PD12:AF09:V, PD11:AF09:V, PE02:AF09:V"

	# mmw command: "memory modify word, modify only given bits"
	# usage: mmw "address setbits clearbits"

	# PB06:AF10:V, PB02:AF09:V, PD13:AF09:V, PD12:AF09:V, PD11:AF09:V, PE02:AF09:V
	# Port B: PB06:AF10:V, PB02:AF09:V
	mmw 0x58020400 0x00002020 0x00001010    ;# MODER
	mmw 0x58020408 0x00003030 0x00000000    ;# OSPEEDR
	mmw 0x5802040C 0x00000000 0x00003030    ;# PUPDR
	mmw 0x58020420 0x0A000900 0x05000600    ;# AFRL
	# Port D: PD13:AF09:V, PD12:AF09:V, PD11:AF09:V
	mmw 0x58020C00 0x0A800000 0x05400000    ;# MODER
	mmw 0x58020C08 0x0FC00000 0x00000000    ;# OSPEEDR
	mmw 0x58020C0C 0x00000000 0x0FC00000    ;# PUPDR
	mmw 0x58020C24 0x00999000 0x00666000    ;# AFRH
	# Port E: PE02:AF09:V
	mmw 0x58021000 0x00000020 0x00000010    ;# MODER
	mmw 0x58021008 0x00000030 0x00000000    ;# OSPEEDR
	mmw 0x5802100C 0x00000000 0x00000030    ;# PUPDR
	mmw 0x58021020 0x00000900 0x00000600    ;# AFRL

	# OCTOSPI1: memory-mapped 4-line read mode with 3-byte(24bits) addresses
	mww 0x52005130 0x00001000				;# OCTOSPI_LPTR: deactivate CS after 4096 clocks when FIFO is full
	# Enter Memory mapped mode 
	mww 0x52005000 0x3040000B				;# OCTOSPI_CR: FMODE=0x11, APMS=1, FTHRES=0, FSEL=0, DQM=0, TCEN=0
	mww 0x52005008 0x00160100				;# OCTOSPI_DCR1: MTYP=0x0, FSIZE=0x16=22=2^(22+1), CSHT=0x00, CKMODE=0, DLYBYP=0
	mww 0x5200500C 0x00000001				;# OCTOSPI_DCR2: WRAPSIZE=0x00, PRESCALER=0+1

	mww 0x52005108 0x00000008				;# OCTOSPI_TCR: SSHIFT=0, DHQC=0, DCYC=0x8
	mww 0x52005100 0x03002303				;# OCTOSPI_CCR: SIOO=0, DMODE=011, ABMODE=0x0, ADSIZE=10, ADMODE=011, ISIZE=0x0, IMODE=011
	mww 0x52005110 0x000000EB				;# OCTOSPI_IR: INSTR=FastRead, 0xeb

	flash probe $a							;# load configuration from CR, TCR, CCR, IR register values
}

# RCC时钟配置，然后调用octospi_init
$_CHIPNAME.cpu0 configure -event reset-init {
	global OCTOSPI1
	global OCTOSPI2

	mmw 0x52002000 0x00000004 0x0000000B	;# FLASH_ACR: 4 WS for  64MHZ HCLK

	mmw 0x58024400 0x00000001 0x00000018	;# RCC_CR: HSIDIV=1, HSI on
	mww 0x58024418 0x00000040				;# RCC_CDCFGR1: CDCPRE=1, CDPPRE=2, HPRE=1
	mww 0x5802441C 0x00000440				;# RCC_CDCFGR2: CDPPRE2=2, CDPPRE1=2
	mww 0x58024420 0x00000040				;# RCC_SRDCFGR: SRDPPRE=2
	mww 0x58024428 0x00404040				;# RCC_PLLCKSELR: DIVM3=4, DIVM2=4, DIVM1=4, PLLSRC=HSI
	mww 0x5802442C 0x01ff0ccc				;# RCC_PLLCFGR: PLLxRGE=8MHz to 16MHz, PLLxVCOSEL=wide
	mww 0x58024430 0x01010207				;# RCC_PLL1DIVR: 64MHz: DIVR1=2, DIVQ1=2, DIVP1=2, DIVN1=8
	mww 0x58024438 0x01010207				;# RCC_PLL2DIVR: 64MHz: DIVR2=2, DIVQ2=2, DIVP2=2, DIVN2=8
	mww 0x58024440 0x01010207				;# RCC_PLL3DIVR: 64MHz: DIVR3=2, DIVQ3=2, DIVP3=2, DIVN3=8
	mmw 0x58024400 0x01000000 0				;# RCC_CR: PLL1ON=1
	sleep 1
	mmw 0x58024410 0x00000003 0				;# RCC_CFGR: PLL1 as system clock
	sleep 1

	adapter speed 4000

	if { $OCTOSPI1 } {
		octospi_init 0
	}
}
```

OpenOCD官方有提供根据你的Pin的配置自动生成SPI GPIO配置的工具，在官方库的`gpio_gen.pl`文件中，需要perl执行。这样就不用对照着手册一个一个地自己去算寄存器的值了，非常的方便。
