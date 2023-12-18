---
title: "PCB - Design 1"
author: "Haobo Gu"
tags: [keyboard]
date: 2022-08-28T21:09:08+08:00
draft: true
summary: Learn PCB Design 
---
# PCB Design - 1
PCB设计的步骤：

1. 原理图设计
2. 原理图整理
3. PCB布局
4. PCB走线
5. 检查设计优化
6. PCB打样以及SMT贴片



## STM32的布线设计

## 

### 键盘布线设计

网格设置： 0.79375mm



## 原理图设计

在原理图设计之前，需要做以下几件事情：

1. 了解主控芯片
2. 分析电路功能

   1. 工作电压

   2. 设计下载电路


## EDA

裸露的线：复制一根线，然后选择顶层（底层）阻焊层


### 连线

要想在多个子图里面连接对应的名称，在右上角添加**网络标签**即可

## Keyboard

### Fancer 65

#### 绘制PCB

矩阵：

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220828232122065.png" alt="image-20220828232122065" style="zoom: 67%;" />





["~\n`","!\n1","@\n2","#\n3","$\n4","%\n5","^\n6","&\n7","*\n8","(\n9",")\n0","_\n-","+\n=",{w:2},"Backspace"],
[{w:1.5},"Tab","Q","W","E","R","T","Y","U","I","O","P","{\n[","}\n]",{w:1.5},"|\n\\"],
[{w:1.75},"Caps Lock","A","S","D","F","G","H","J","K","L",":\n;","\"\n'",{w:2.25},"Enter"],
[{w:2.25},"Shift","Z","X","C","V","B","N","M","<\n,",">\n.","?\n/",{w:2.75},"Shift",{x:1.25},"↑"],
[{w:1.25},"Ctrl",{w:1.25},"Win",{w:1.25},"Alt",{a:7,w:6.25},"",{a:4,w:1.25},"Alt",{w:1.25},"Win",{w:1.25},"Menu",{w:1.25},"Ctrl",{x:0.25},"←","↓","→"]
