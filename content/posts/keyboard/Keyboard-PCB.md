---
title: "Keyboard PCB design"
author: "Haobo Gu"
tags: [keyboard]
date: 2022-07-28T16:19:23+08:00
summary: Learn Ai03's keyboard PCB design guide
draft: true
---
# Draw

Ai03's guide for making a keyboard pcb

## IDE

Ai03 uses KiCad for drawing pcbs. We'll use [jialichuang's web eda editor](https://pro.lceda.cn/), which is much more convenient.

## Let's start

First, you should add your MCU to the sheet. I'd use stm32f411ceu6 as an example:

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220831145204665.png" alt="image-20220831145204665" style="zoom:50%;" />

You'll see there are many IOs. But how to set it up? We'll need its [datasheet](https://www.st.com/resource/en/datasheet/stm32f303vc.pdf). The datasheet provides specs and a lot of useful information about the MCU. If you are using other MCUs, just search on the internet and get corresponding datasheet.

### Make MCU work

There are necessary steps to make mcu work. We'll go through it one by one.

### Power supply

First of all we should provide the MCU with power. According to the datasheet, stm32f411ce needs 3.3V voltage as power supply and some capacitors is need. Because USB provides 5V voltage, we have to convert it to 3.3V using a voltage converting chip(here we used `ams1117-3.3`):

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220831145533118.png" alt="image-20220831145533118" style="zoom:50%;" />

stm32f411 also needs some other capacitors for filtering. There is the demo power supply scheme:

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220831150024265.png" alt="image-20220831150024265" style="zoom:50%;" />

You can also learn other open source project for power supply module. Here we add some capacitors as following:

<img src="https://haobogu-md.oss-cn-hangzhou.aliyuncs.com/markdown/imgs/image-20220831150205673.png" alt="image-20220831150205673" style="zoom:50%;" />













































