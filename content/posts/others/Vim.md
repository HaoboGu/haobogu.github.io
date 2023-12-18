---
title: "Vim 命令"
author: "Haobo Gu"
tags: [vim]
date: 2019-08-15T22:16:15.380737+08:00
---

<!--more-->
## 常用命令

### Normal Mode

| 按键          | 含义           |
| ------------- | -------------- |
| >>            | 向右缩进       |
| <<            | 向左缩进       |
| ctrl w + hjkl | 跳转窗口到方向 |
| ctrl w + w    | 下一个窗口     |

### Command Mode

| 命令 | 含义         |
| ---- | ------------ |
| :sp  | 上下拆分窗口 |
| :vsp | 左右拆分窗口 |

## 搜索和替换

替换的语法：`范围s/original_text/replacement/替换选项`

替换当前行：`s/orginal_text/replacement/g`，`g`为global全局替换，即替换范围内的所有目标

#### 替换范围

替换所有行只需要在前面加个`%`，即`%s/orginal_text/replacement/g`

选择区域替换：先`ctrl+v`进入Visual Mode，选择完区域后输入`:`进入命令行模式，范围就自动添加了，后面一样。也可以手动指定，如：`2,12s/original_text/replacement/g`搜索2到12行；`.,+5s/original_text/replacement/g`表示搜索当前行`.`和接下来5行`+5`

#### 替换选项

`g`：global替换

`i/I`：大小写不敏感/敏感

`c`：需要确认

## CoC

一般来说，在coc.nvim中，tab选择，回车或者继续打字确定

一些其他的增强功能如下表：

注意，其中`\`为`<leader>`键，在vimrc中，`<leader>r`即为`\r`。Leader键一般用来表示自定义的一些快捷键

| 按键 | 含义             |
| ---- | ---------------- |
| gd   | 转到定义         |
| gy   | 转到类型的定义   |
| gi   | 转到实现         |
| gr   | 转到reference    |
| \r   | 替换下一个单词   |
| \f   | format选定的区域 |



