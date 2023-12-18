---
title: "Basics of frontend development"
author: "Haobo Gu"
tags: [frontend]
date: 2022-04-14T19:57:40+08:00
summary: 学学前端开发：HTML + CSS
draft: true
---

# Basics of frontent development
本文介绍了前端开发所需要的最基础的知识。大家都知道，前端开发三大件：HTML + CSS + JavaScript。在本文中我会先介绍HTML + CSS。

## HTML
### HTML简介
什么是HTML？简单来说，HTML是标记网页的结构、内容用的，它定义了一个网页的内容结构。HTML由一系列元素（element）组成，下面是一个简单的元素：
```html
<p>Hello World</p>
```
这个元素代表着在网页上面渲染的一个元素，它包含两个主要内容：标签(`<p></p>`)和文本内容。标签中包含着元素的名称，在例子中为p，即paragraph。而标签中也可以含一些属性，比如下面的标签就有一个属性名为class：
```html
<p class="editor-note">Hello World</p>
```
下面是一个完整的网页例子：
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>测试页面</title>
  </head>
  <body>
    <img src="images/firefox-icon.png" alt="测试图片">
  </body>
</html>
```
每一行每一个标签的解释可以看https://developer.mozilla.org/zh-CN/docs/Learn/Getting_started_with_the_web/HTML_basics#html_%E6%96%87%E6%A1%A3%E8%AF%A6%E8%A7%A3

HTML中存在着各种各样的标签，这些标签就标记了所有网页内容元素的类型。下面的章节会介绍一下常见的HTML标签

### 常见HTML标签
#### 标记文本内容的标签

- `<h1>`: 标题，其中数字代表标题层级，1代表最高层级
- `<h2>`: 同上
- `<p>`: 段落（paragraph），即正文文本
#### 列表标签：
- `<ul>`: 无序列表（unordered list）
- `<ol>`: 有序列表（ordered list）
- `<li>`: 列表的元素（list item）
下面是一个例子：
```html
<p>我们是谁？</p>

<ul>
  <li>技术人员</li>
  <li>思考者</li>
  <li>建造者</li>
</ul>

<p>我们致力于……</p>
```
#### 链接
我们使用`<a>`表示一个链接，a是archor（锚点）的缩写。要想让链接有自动跳转的功能，需要给`<a>`标记的标签设置`href`属性：
```html
<a href="https://haobogu.github.io">点击进入我的Blog</a>
```
#### 图片
在HTML中，使用<img>来表示一个图片，图片的地址使用`src`属性：
```html
<img src="images/xxx.jpg" alt="测试图片">
```

HTML基础一共就这么多，看完了就大概可以看懂一个HTML的结构了，更复杂的内容后续再说，接下来我们介绍CSS。

## CSS
CSS的全称是Cascading Style Sheet，即层叠样式表。它是用来为网页添加各种样式的代码，比如设置某些字体的颜色、大小，设置网页的行间距等等。

### CSS简介
CSS可以用来选择性地为HTML元素添加样式，举个例子，如果我们想把HTML页面中所有<p>元素的文本设置为红色，那么可以使用如下CSS代码：
```css
p {
    color: red;
}
```
然后，在HTML代码中添加如下一行，通知网页使用我们自己的css样式文件：
```HTML
<link href="path/to/your.css" rel="stylesheet">
```
That's it! 我们就完成了使用CSS样式文件对HTML网页进行修改

### CSS语法
上面我们定义的CSS非常简单，但是其中包含了好几个部分，如下图：
<img src="https://mdn.mozillademos.org/files/16483/css-declaration.png">

整个CSS内容我们称之为**规则集**，意为HTML展示的规则（我瞎掰的）。其中，选择器Selector表示了当前的规则生效的HTML元素类型，然后大括号中的每一条都是一个声明，用来指定对应样式元素的属性。这些属性可以包含字体字号颜色等等。

每一个规则也可以对多种HTML标签生效，只需要修改选择器即可：
```css
p, li, h1 {
    color: red;
}
```
### 选择器
选择器声明了当前CSS规则定义的样式对哪些元素生效，除了上面使用的元素选择器之外，CSS还提供了如下几种方式定义选择器：
1. ID选择器：选择具有特定ID的元素，用一个`#`作为前缀代表类选择器
```css
#my-id {
    color: red;
}
```
这种选择器生效的HTML元素长这样：
```html
<p id="my-id">xxx</p>
```
2. 类选择器：选择特定class的元素，用一个`.`作为前缀代表类选择器
```css
.my-class {
    color: red;
}
```
这种选择器生效的HTML元素长这样：
```html
<p class="my-class">xxx</p>
```
3. 属性选择器：选择**拥有特定属性**的元素：
```css
p[src] {
    color: red;
}
```
上面的样式只会对拥有`src`属性的`<p>`标签生效：`<p src=""></p>`。这种选择器还可以用属性具体的值（支持正则！）做过滤，如下面的选择器只会选择`src`的值为`abc`的元素：
```css
p[src=abc] {
  color: red;
}
```
这里的属性值的选择，可以有如下的几种可能：
```
a=b 属性a的值绝对等于b
a*=b 属性a包含b
a^=b 属性a的值以b开头
a$=b 属性a的值以b结尾
```

4. 伪（Pseudo）类选择器：选择特定状态下的特定元素，后面加`:`
```css
p:hover {
    color: red;
}
```
仅在鼠标指针悬停时选择

#### More selectors
实际上，CSS中还有其他一些选择器（相对较少用到），为元素选择提供了丰富的选项，如：

第一大类，层次选择器，顾名思义就是根据HTML元素的层次进行选择，有如下一些：
1. 后代选择器

```css
body p {
  color: red;
}
```
对`<body>`的所有后代生效。

2. 子选择器
```css
body>p {
  color: red;
}
```
只对`<body>`的儿子那一层的HTML元素生效。

3. 相邻兄弟选择器（只选择下面的一个兄弟）
```css
.active + p {
  color: red;
}
```
对`class='active'`的同级的下一个`<p>`生效。

4. 通用选择器（选择下面所有兄弟）
```css
.active~p {
  color: red;
}
```
对`class='active'`后面的所有`<p>`生效。


### CSS盒模型
CSS 布局主要就是基于盒模型的。它把页面的每一个部分都看做是一个方块，每个占据页面空间的块都有这样的属性：
- padding：即内边距，围绕着内容（比如段落）的空间。
- border：即边框，紧接着内边距的线。
- margin：即外边距，围绕元素外部的空间。
<img src="https://mdn.mozillademos.org/files/9443/box-model.png">
具体设置的例子参考：https://developer.mozilla.org/zh-CN/docs/Learn/Getting_started_with_the_web/CSS_basics#%E6%96%87%E6%A1%A3%E4%BD%93%E6%A0%BC%E5%BC%8F%E8%AE%BE%E7%BD%AE

