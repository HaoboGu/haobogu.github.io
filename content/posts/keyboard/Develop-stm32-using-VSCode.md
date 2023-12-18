---
title: "Develop stm32 using VSCode + PlatformIO on MacOS"
author: "Haobo Gu"
tags: [keyboard]
date: 2022-07-29T19:57:40+08:00
summary: 在Mac上面使用VSCode开发stm32
---
# 在Mac上面使用VSCode开发stm32

因为最近有项目需要对stm32进行硬件的开发，而我现在主要是在用M1芯片的Mac，且官方提供的各种配套工具都不太行，所以记录一下在Mac下折腾stm32开发的过程。

## 准备

首先，进行硬件开发你得有相应的硬件。我是从淘宝上买了stm32f411ceu6的开发板，以及st-link v2的烧写器，在本文中我们就以它们作为示例。

其次，需要下载宇宙第一编辑器VSCode，并且安装PlatformIO插件。这个过程就不再展开了，可能安装PlatformIO插件会慢点，等待即可。

### 安装STM32CubeMX

再然后就需要安装STM32CubeMX了。这个工具是用来自动生成对应的工程的，要开发stm32系列单片机，几乎也是必须的。

1. 首先，下载STM32CubeMX安装包：https://www.st.com/zh/development-tools/stm32cubemx.html#overview。官方提供了Mac系统对应的安装包，点击下载即可，大约几百M

2. 下载下来是一个zip文件，解压之

3. 解压后的文件夹中，有一个`.app`文件。一般来说直接执行这个.app文件即可，但是对于M1的Mac来说，由于苹果的安全策略，导致直接安装不可行，需要使用如下命令处理`.app`文件

   ```shell
   sudo xattr -cr xxxxx.app
   ```

4. 安装过程需要你本地装有Java，如果你本地没有Java，或者Java版本不对，依然不行。不过好在在安装包中，已经贴心地给你准备好了Jre（没错，压缩包解压之后有一个jre文件夹，就是做这个的）。不过，由于苹果的安全策略，你还是没有办法直接使用`jre`文件夹中的Java，类似地，你需要对jre文件夹再执行一遍`xattr`：

   ```shell
   sudo xattr -cr jre
   ```

5. 现在你可以使用Java执行安装包：

   ```shell
   java -jar xxx.app
   ```

6. 之后，会弹出安装包的界面（这界面真复古）。一路next，就能可以把STM32CubeMX安装好了

## 使用STM32CubeMX初始化工程

所有的依赖都安装好了，下面来初始化工程。打开STM32CubeMX，上面的File菜单中选择New Project（可能会读条，等待即可）。然后在新建工程的窗口中搜索你的MCU，我这里使用stm32f411ceu6：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220729182329843.png" alt="image-20220729182329843" style="zoom:50%;" />

选择完MCU之后，右上角Start Project，就进入初始化页面。前面的选项都不用改，直接点到Project Manager部分，输入一个Project名称，然后在**ToolChain/IDE这里选择Makefile**：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220729182744493.png" alt="image-20220729182744493" style="zoom:50%;" />

点击右上角GENERATE CODE即可生成代码

## Option1：使用PlatformIO

打开VSCode的命令界面，输入show platformIO，打开platformIO的主页面，点击页面上的New Project创建工程。在创建工程弹窗中， Project Name填写刚刚在CubeMX中写的Project名称，Board选择stm32f411CE，framework注意需要选择STM32Cube。然后重点来了，取消Use default location勾选，并且在下面的路径选择中选择**刚刚CubeMX生成的代码目录的父文件夹**。

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220729183529388.png" alt="image-20220729183529388" style="zoom:50%;" />

点击finish，初次生成可能需要等待，完成之后VSCode会自动跳到对应的文件夹。

然后，检查一下文件下的根目录下面是否同时存在`[你的工程名].ioc`和`platformio.ini`，如果有，那说明代码工程已经被成功创建。

接下来，打开`platformio.ini`，把CubeMX生成的Src目录和Header目录设置为platformIO的对应目录，然后设置一下下载器，我使用的是st-link：

```ini
# platformio.ini

[env:genericSTM32F411CE]
platform = ststm32
board = genericSTM32F411CE
framework = stm32cube
# 设置下载器
debug_tool = stlink
upload_protocol = stlink

[platformio]
include_dir=Core/Inc
src_dir=Core/Src
```

Done！所有的配置都完成了。当然你也可以把platformIO自动生成的工程文件夹`src`, `lib `, `test`删掉（我们用的是CubeMX生成的代码工程，不是吗）。

### 烧录和调试

连接st-link以及开发板，单击VSCode下面的upload选项即可烧录代码：

![image-20220729184552432](https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220729184552432.png)

如果想要调试的话，单击F5（VSCode的默认调试快捷键）即可。Easy， huh？

## Option2：使用makefile + OpenOCD

如果不想使用platformIO，其实也是可以的。只不过需要对VSCode多一点点配置。

### C/C++设置

在生成工程之后，首先打开`.vscode`目录下的`c_cpp_properties.json`，新建一个配置项，比如`Mac`，然后把各种依赖添加进`includePath`，并且设置`browse`、`defines`、`compilerPath`等参数：

```json
{
    "configurations": [
        {
            "name": "Mac",
            "includePath": [
                "${workspaceFolder}/**",
                "${workspaceFolder}/Inc",
                "${workspaceFolder}/Drivers/STM32H4xx_HAL_Driver/Inc",
                "${workspaceFolder}/Drivers/STM32H4xx_HAL_Driver/Inc/Legacy",
                "${workspaceFolder}/Drivers/CMSIS/Include",
                "${workspaceFolder}/Drivers/CMSIS/Device/ST/STM32F4xx/Include",
                "/opt/homebrew/Cellar/arm-none-eabi-gcc/10.3-2021.07/gcc/arm-none-eabi/include",
                "/opt/homebrew/Cellar/arm-none-eabi-gcc/10.3-2021.07/gcc/arm-none-eabi/include/c++/10.3.1",
                "/opt/homebrew/Cellar/arm-none-eabi-gcc/10.3-2021.07/gcc/lib/gcc/arm-none-eabi/10.3.1/include",
                "/opt/homebrew/Cellar/arm-none-eabi-gcc/10.3-2021.07/gcc/lib/gcc/arm-none-eabi/10.3.1/include-fixed"
            ],
            "defines": [
                "USE_HAL_DRIVER",
                "STM32F411xE"
            ],
            "macFrameworkPath": [
                "/Library/Developer/CommandLineTools/SDKs/MacOSX12.sdk/System/Library/Frameworks"
            ],
            "compilerPath": "/opt/homebrew/bin/arm-none-eabi-gcc",
            "cStandard": "c17",
            "cppStandard": "c++20",
            "intelliSenseMode": "macos-clang-arm64",
            "configurationProvider": "ms-vscode.makefile-tools",
            "browse": {
                "path": [
                    "${workspaceFolder}/**",
                    "${workspaceFolder}/Inc",
                    "${workspaceFolder}/Drivers/STM32H4xx_HAL_Driver/Inc",
                    "${workspaceFolder}/Drivers/STM32H4xx_HAL_Driver/Inc/Legacy",
                    "${workspaceFolder}/Drivers/CMSIS/Include",
                    "${workspaceFolder}/Drivers/CMSIS/Device/ST/STM32F4xx/Include",
                    "/opt/homebrew/Cellar/arm-none-eabi-gcc/10.3-2021.07/gcc/arm-none-eabi/include",
                    "/opt/homebrew/Cellar/arm-none-eabi-gcc/10.3-2021.07/gcc/arm-none-eabi/include/c++/10.3.1",
                    "/opt/homebrew/Cellar/arm-none-eabi-gcc/10.3-2021.07/gcc/lib/gcc/arm-none-eabi/10.3.1/include",
                    "/opt/homebrew/Cellar/arm-none-eabi-gcc/10.3-2021.07/gcc/lib/gcc/arm-none-eabi/10.3.1/include-fixed"
                ],
                "limitSymbolsToIncludedHeaders": true,
                "databaseFilename": "${workspaceFolder}/.vscode/browse.vc.db"
            }
        }
    ],
    "version": 4
}
```

需要注意的是，`defines`需要和你的`Makefile`里面的C defines对应

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220913154311025.png" alt="image-20220913154311025" style="zoom:33%;" />

另外， 添加进Path中的`arm-none-eabi-gcc`需要填写你本地的对应SDK的安装地址。

然后，我们可以在命令行中执行`make`来测试是否可以正确编译。注意，如果你添加了额外的`.c`文件，则需要在Makefile里面指定：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220913155123648.png" alt="image-20220913155123648" style="zoom:50%;" />

如果你看到了如下信息，则说明工程配置基本没有问题：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220913155232390.png" alt="image-20220913155232390" style="zoom:50%;" />

### OpenOCD配置

首先安装OpenOCD：

```shell
brew install openocd
```

安装完毕之后，使用`openocd --version`检查是否安装成功。

然后，在项目根目录下创建`openocd.cfg`，并且参考`/opt/homebrew/Cellar/open-ocd/0.11.0/share/openocd/scripts/`下的配置，配置你自己的芯片。比如，对于我使用的stm32f411 + stlink，我需要进行如下配置：

```
# Use stlink
source [find interface/stlink.cfg]

transport select hla_swd

# Set target
source [find target/stm32f4x.cfg]
```

#### 注意

在Windows下面，OpenOCD配置有个坑就是，我们需要使用sysprogs公司编译的OpenOCD，原因是VSCode中cortex-debug插件默认使用了VirtualGDB，而VirtualGDB就是sysprogs公司出品的。因此我们还需要使用该公司编译的OpenOCD才能正确下载。我们可以去[这里](https://sysprogs.com/getfile/1364/openocd-20201228.7z)下载对应的OpenOCD，解压并且添加到Path环境变量中。

### 添加SVD文件

添加`.svd`文件可以在调试的时候看到各个外设的状态，非常方便。首先在这里下载对应芯片的svd文件：https://github.com/posborne/cmsis-svd/

如果这里没有，可以去keil的官网（https://www.keil.com/dd2/pack/）下载对应开发包，解压之后去`CMSIS/SVD`目录下找对应的SVD（网页很卡，小心）。

把SVD复制到项目根目录备用。

### VSCode设置

接下来就是VSCode的debugger设置。首先，需要配置编译命令。在`.vscode`文件夹下面添加`tasks.json`，并且填入如下内容：

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "make all",
            "group": "build",
            "type": "shell",
            "command": "make",
            "args": [
                "all",
                "-j",
                "4"
            ]
        },
        {
            "label": "make clean",
            "group": "build",
            "type": "shell",
            "command": "make",
            "args": [
                "clean"
            ]
        }
    ]
}
```

VSCode的task相关内容可以去看VSCode官方文档，介绍得非常详细。这里我们就定义了两个task，一个是`make all`，一个是`make clean`。

在配置完Task之后，按下快捷键`cmd+shift+b`，就可选择快速运行task了：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220913155706281.png" alt="image-20220913155706281" style="zoom:50%;" />

然后，我们配置debug。首先安装`cortex-debug`插件。安装完毕之后，在`.vscode`目录下创建`launch.json`，并且进行如下配置：

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Cortex Debug",
            "cwd": "${workspaceFolder}",
            "executable": "./build/f411.elf",
            "request": "launch",
            "type": "cortex-debug",
            "runToEntryPoint": "main",
            "servertype": "openocd",
            "showDevDebugOutput": "parsed",
            "configFiles": [
                "openocd.cfg"
            ],
            "svdFile": "STM32F411xx.svd",
            "device": "stlink",
            "preLaunchTask": "make all"
        }
    ]
}
```

需要注意的是，在`executable`下，需要填写你自己的固件路径。然后设置`svdFile`为你刚刚下载的那个SVD文件。然后点击F5就可以愉快地debug了。在设置了SVD之后，还可以在Debug页面下看到各个外设和寄存器的状态：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220913162207367.png" alt="image-20220913162207367" style="zoom:50%;" />

## IAP

在产出的`.bin`文件末尾追加4bytes的CRC校验值，然后在bootloader中，首先从USB读取文件到一块Flash的临时区域，然后根据文件的长度，计算`.bin`文件的CRC校验和并且和`.bin`文件末尾的值作比较，如果一样，则正式把结果烧写到Flash的程序运行区域。否则直接退出。这样可以防止在复制固件的过程中出现错误，从而让系统变砖。























