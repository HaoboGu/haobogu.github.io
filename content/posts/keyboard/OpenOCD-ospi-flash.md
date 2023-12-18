---
title: "使用OpenOCD+VSCode一键烧录Boot+App到内置+外置flash"
author: "Haobo Gu"
tags: [keyboard, stm32, embedding]
date: 2022-10-31T11:53:23+08:00
summary: OpenOCD + gcc + VSCode, YES!
---

# 使用OpenOCD+VSCode一键烧录Boot+App到内置+外置flash

## 背景

在开发stm32系列的时候，大多数情况下会使用Windows系统+MDK/IAR来开发。不过对于主力是MacOS+Windows的我来说，一套兼容两个系统的开发方案就成了刚需。在中文互联网上面类似的资料非常稀少，不过实际上，使用gcc+OpenOCD的方案实际上已经非常成熟了，在此就记录一下我的折腾历程。

## 最终效果

在讲具体配置步骤之前，先看一下最终的效果：

VSCode中，`shift+cmd+b`，选择`flash merged firmware`，然后VSCode会帮你把搞定以下所有：

1. 自动编译App+Bootloader两个工程
2. 使用srec_cat合并bootloader.hex和application.hex（同时兼容MacOS和Windows）
3. 把bootloader.hex烧写到stm32内置Flash，把application.hex烧写到外置OSPI Flash

一键搞完，相当爽：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221028110201986.png" alt="image-20221028110201986" style="zoom:50%;" />

如果你选择`flash bootloader`或者`flash application`，也一样，VSCode会首先编译对应的hex，然后自动烧录到对应Flash。

### Debug

按F5就可以开始debug，如果默认配置的是Debug Bootloader，那么会先编译烧录bootloader，然后开始debug；Debug Application也一样。且VSCode会自动地在bootloader和application之间跳转：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221028111055464.png" alt="image-20221028111055464" style="zoom:30%;" />

## 开发环境配置

首先，开发环境的配置可以参考这里：https://haobogu.github.io/posts/keyboard/develop-stm32-using-vscode/。需要注意的是我们需要选择gcc+makefile的方案而不是PlatformIO。

需要注意的是，如果你想要安装最新版的OpenOCD，在Mac上面可以直接使用

```
brew install openocd --HEAD
```

另外，还需要使用homebrew安装srecord备用：

```
brew install srecord
```

在Windows下，这两者都需要手动安装，srecord需要一个`srec_cat.exe`。

## 创建工程

由于我们的工程是App + Bootloader形式的，因此需要在根目录下创建两个工程。创建完之后，可以把打包编译都加到VSCode的task里。然后，直接使用`shift+cmd+b`快捷键就可以选择任务：

**<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221028105511254.png" alt="image-20221028105511254" style="zoom:40%;" />**

下面是一个我的task.json，供参考

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "make bootloader",
            "group": "build",
            "type": "shell",
            "command": "cd bootloader && make all -j 16",
            "problemMatcher": {
                "owner": "cpp",
                "fileLocation": [
                    "relative",
                    "${workspaceFolder}/application"
                ],
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            }
        },
        {
            "label": "make application",
            "group": "build",
            "type": "shell",
            "command": "cd application && make all -j 16",
            "problemMatcher": {
                "owner": "cpp",
                "fileLocation": [
                    "relative",
                    "${workspaceFolder}/application"
                ],
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning|error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            },
        },
        {
            "label": "make all",
            "group": "build",
            "type": "shell",
            "command": "cd application && make all -j 16 && cd ../bootloader && make all -j 16"
        },
        {
            "label": "make clean",
            "group": "build",
            "type": "shell",
            "command": "cd bootloader && make clean && cd ../application && make clean"
        },
        {
            "label": "flash bootloader",
            "group": "build",
            "type": "shell",
            "command": "cd bootloader && openocd -f openocd.cfg -c \"program build/bootloader.elf preverify verify reset exit\"",
            "dependsOn": [
                "make bootloader"
            ],
            "dependsOrder": "sequence"
        },
        {
            "label": "flash application",
            "group": "build",
            "type": "shell",
            "command": "cd application && openocd -f openocd.cfg -c \"program build/application.hex preverify verify reset exit 0x00000000\"",
            "dependsOn": [
                "make application"
            ],
            "dependsOrder": "sequence"
        },
        {
            "label": "flash merged firmware",
            "group": "build",
            "type": "shell",
            "command": "openocd -f bootloader/openocd.cfg -c \"program firmware.hex preverify verify reset exit\"",
            "dependsOn": [
                "make all",
                "merge hex"
            ],
            "dependsOrder": "sequence"
        },
        {
            "label": "merge hex",
            "group": "none",
            "type": "shell",
            "command": "resources/srec_cat.exe",
            "args": [
                "bootloader/build/bootloader.hex",
                "-Intel",
                "application/build/application.hex",
                "-Intel",
                "-o",
                "firmware.hex",
                "-Intel"
            ],
            "osx":{
                // needs `brew install srecord`
               "command": "srec_cat",
               "args": [
                    "bootloader/build/bootloader.hex",
                    "-Intel",
                    "application/build/application.hex",
                    "-Intel",
                    "-o",
                    "firmware.hex",
                    "-Intel"
                ]
            },
            "problemMatcher": []
        }
    ]
}
```

### Debug

Debug的配置也类似，只不过是配置`launch.json`，具体可以参考上面环境配置的文章。下面是我的`launch.json`，供参考：

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug Bootloader",
            "cwd": "${workspaceFolder}",
            "executable": "bootloader/build/bootloader.elf",
            "loadFiles": [
                "bootloader/build/bootloader.elf",
                "application/build/application.elf"
            ],
            "symbolFiles": [
                {
                    "file": "bootloader/build/bootloader.elf",
                },
                {
                    "file": "application/build/application.elf"
                }
            ],
            "request": "launch",
            "type": "cortex-debug",
            "runToEntryPoint": "main",
            "servertype": "openocd",
            "showDevDebugOutput": "parsed",
            "configFiles": [
                "bootloader/openocd.cfg"
            ],
            "svdFile": "bootloader/STM32H7B0x.svd",
            "device": "stlink",
            "preLaunchTask": "flash bootloader"
        },
        {
            "name": "Debug Application",
            "cwd": "${workspaceFolder}",
            "executable": "bootloader/build/bootloader.elf",
            "loadFiles": [
                "bootloader/build/bootloader.elf",
                "application/build/application.elf"
            ],
            "symbolFiles": [
                {
                    "file": "bootloader/build/bootloader.elf",
                },
                {
                    "file": "application/build/application.elf"
                }
            ],
            "request": "launch",
            "type": "cortex-debug",
            "runToEntryPoint": "main",
            "servertype": "openocd",
            "showDevDebugOutput": "parsed",
            "configFiles": [
                "bootloader/openocd.cfg"
            ],
            "svdFile": "bootloader/STM32H7B0x.svd",
            "device": "stlink",
            "preLaunchTask": "flash application"
        }
    ]
}
```

## Linker Script配置

由于我们的application运行在外置的OSPI Flash，要想让OpenOCD默认把application烧录到OSPI Flash，需要首先修改一下Linker Script。其实修改也非常简单，就是把Flash区域的地址由修改到0x90000000，大小修改为4096K（即Flash的大小设置为了4M，这个可以根据你使用的spi flash芯片来定）。

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221028112019836.png" alt="image-20221028112019836" style="zoom:35%;" />

为什么需要把Flash区域的地址修改到0x90000000呢？其实这一点和使用MDK/IAR并没有什么不同，简单来说就是stm32要想在直接在OSPI内运行（即XIP），那么OSPI必须运行在memory mapped模式下。而memory mapped模式下的默认地址就是0x90000000。

注意这里有个坑就是，在OSPI Flash打开memory mapped模式之后，正常的OSPI Flash的读写通信就全都会失败。如果你想使用驱动里面的读写或者其他函数（比如ReadID），那么必须关掉memory mapped模式。

## OpenOCD的配置

下面就来到了重点，也是最难的地方：OpenOCD的配置。

对于大部分程序来说，其实只需要使用OpenOCD官方提供的默认配置就行了。但是，由于我们要在OSPI Flash上面XIP运行程序，官方的配置大概率是不能用的，就需要我们自己写`openocd.cfg`。

在`openocd.cfg`中，实际上需要做以下几件事：

1. 指定debugger，在这里我们使用的是stlink
2. 指定芯片，source芯片的基础配置
3. 配置时钟
4. 配置GPIO
5. 配置ospi

其中，1和2都比较简单，不再展开，参考下面的代码即可。那为什么在OpenOCD中还需要配置时钟、GPIO和OSPI呢？这是因为我们想要直接把application烧录到OSPI Flash，在OpenOCD烧录的过程中，它需要知道你的OSPI的GPIO配置，以及OSPI Flash的配置，这样OpenOCD才能正确地和OSPI Flash通信，从而进行烧录。

3/4/5的配置，本质上就是配置stm32的寄存器。在OpenOCD中可以使用`mww`和`mmw`对寄存器进行设置，`mww`命令在OpenOCD文档里面就有，而`mmw`是OpenOCD额外封装的一个命令，具体逻辑可以去OpenOCD的`scripts`里面找到。配置代码如下：

```tcl
、
# Choose debugger
source [find interface/stlink.cfg]

# Choose transport interfance 
transport select hla_swd

# Set chip name
set CHIPNAME stm32h7b0xx

# Enable stmqspi
if {![info exists OCTOSPI1]} {
	set OCTOSPI1 1
	set OCTOSPI2 0
}

# Use built-in stm32h7 openocd configs
source [find target/stm32h7x.cfg]

# OCTOSPI initialization
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

	sleep 1

	flash probe $a							;# load configuration from CR, TCR, CCR, IR register values
}

# RCC配置，然后调用octospi_init
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

reset_config none separate
```

首先看`$_CHIPNAME.cpu0 configure -event reset-init`部分，这里是时钟的配置。具体哪一条命令配置的是哪个寄存器，注释里面已经很明白了。这里时钟最好和Bootloader里面配置的时钟一致，否则可能出现奇怪的问题（比如OSPI Flash获取不到ID之类的）。

在时钟配置完毕之后，会判断是否开启了`OCTOSPI1`，如果开启，则使用`octospi_init`函数对OSPI进行配置。而OSPI的配置在上面`proc octospi_init { octo }`函数里。

然后就是OSPI的具体配置了。OSPI的配置分两步，GPIO配置和OSPI本身的寄存器配置。对于GPIO配置，OpenOCD官方提供了一个perl脚本可以直接生成对应引脚的GPIO，perl脚本的链接在[这里](https://github.com/openocd-org/openocd/blob/36597636f2a0e9fbf7b44f2c02fb85632d4dc063/contrib/loaders/flash/stmqspi/gpio_conf_stm32.pl)。具体使用可以参考我上面写的注释，需要配置好对应的接口和AF。而OSPI配置没有什么技巧，就是对照着OSPI的寄存器一个一个地配置。需要注意的是，这里的OSPI Flash的配置必须和Bootloader里面的OSPI Flash的配置完全一致，这样的话在烧录完Bootloader之后，OpenOCD才能够正确地使用这个配置去烧写Application到OSPI Flash。总而言之，在Cube里面的配置、OpenOCD的配置和Flash的配置最好都要一致，才不会出现奇怪的问题。

### 验证配置

在配置完毕OpenOCD之后，可以手动验证一下配置。这一步非常重要，防止后面出问题不知道去哪里排查。首先进入你的application文件夹，然后使用命令打开OpenOCD监听4444端口：

```shell
openocd -f openocd.cfg
```

然后，使用`telnet`(windows/linux)或者`nc`连接到OpenOCD：

```
# Windows/Linux
telnet localhost 4444
# MacOS
nc localhost 4444
```

连接到之后，就可以任意输入OpenOCD的命令了，在这里我们使用

```shell
flash probe 1
```

来查看OSPI Flash的信息（你也可以尝试一下`flash probe 0`，看看输出的信息）：

```
> flash probe 1
flash probe 1
valid SFDP detected
flash1 'sfdp' id = 0x333333 size = 8192 KiB
flash 'stmqspi' found at 0x90000000
```

可以看到OpenOCD已经识别出了在0x90000000位置的stmqspi。然后我们可以使用`flash info 1`来查看Flash的详细信息：

![image-20221031113620913](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20221031113620913.png)

可以验证一下下面的Flash信息配置和你使用的Flash的DataSheet中的配置是否一致。如果一致，那么应该是没有问题的。

看到这里，你可能会有一个问题就是，为什么这里的Flash名称显示的是`sfdp`，而前面烧录的时候会正确显示Flash的名称`w25q64jv`？

这里原理我也没有弄明白，但是我猜测是因为OSPI的Memory mapped模式已经打开，这样的话GetID的命令就会失效。不过这个时候，OpenOCD只需要读取sfdp的寄存器就可以知道用那些命令操作Flash了。

## 编译工程，合并Hex，烧录

由于我们是Bootloader + Application两个工程，在编译的时候需要两个工程一起编译，编译出来是两个固件文件。想要实现一键烧录，有两个方案：

1. 分别烧录两个固件到对应位置
2. 把两个固件合并成一个烧录

其实两个方案没太大区别，为了后续发布简单，我选择了方案2。方案2有一个问题就是，不能使用bin格式的固件。这是因为bin格式在内存上是连续的，而我们的固件实际上是烧录在两个位置：内置Flash（0x08000000)和OSPI Flash（0x90000000）。如果使用bin格式的话，在合并之后，就会产生一个好几个G的超大固件，这显然是不对的。所以在这里我们使用hex格式，hex格式在文件内有存储每一个块烧录的内存位置，因此我们只需要使用三方工具把Application和Bootloader的固件合并成一个，然后在烧录的时候就会自动地把Bootloader烧录到0x08000000，把OSPI Flash烧录到0x90000000。

我们选择的合并工具是`srecord`，也是非常著名的hex合并工具，全平台都有对应版本。安装上面已经讲过，不再赘述。想要合并hex，使用以下命令即可：

```shell
srec_cat bootloader.hex -Intel application.hex -Intel -o merged.hex -Intel
```

代码实际上都写在了VSCode的task中，执行task就会自动完成hex的合并。

烧录就简单了：

```shell
openocd -f bootloader/openocd.cfg -c \"program firmware.hex preverify verify reset exit\"
```

这里需要注意的有两点：

1. 使用的是bootloader文件夹下的openocd.cfg，bootloader和application的openocd.cfg最好保持一致
2. 使用的是OpenOCD的`program`命令，而不是很常见的`flash erase_xxxx`。`program`命令是对`flash`命令的额外一层封装，还提供了预校验、烧写之后的校验、自动退出重启等功能，强烈建议使用。

