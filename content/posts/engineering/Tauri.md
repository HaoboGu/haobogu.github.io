---
title: "Learn Tauri"
author: "Haobo Gu"
tags: [rust]
summary: Learn how to write a desktop software with tauri
date: 2023-06-30T14:26:42.689638+08:00
dradf: true
---

# Learn Tauri

Tauri is a desktop application framework which has a Rust core and can be used with any frontend framework. It's similar with Electron, but tauri uses system's built-in web render, so the application created by tauri is much smaller than Electron.

## Installation

Tauri can be used with any frontend frameworks. We'll use vite in this article. If you don't have a project, use the following to create one:

```shell
cargo install create-tauri-app
cargo create-tauri-app
```

Another thing you need to do is to install tauri cli tool to cargo or npm:

```shell
cargo install tauri-cli
# OR
npm install --save-dev @tauri-apps/cli
```

Then, you can use cargo or npm command like:

```shell
cargo tauri dev
# OR
npm run tauri dev
```

tauri-api npm package is also needed for calling Rust APIs in frontend:

```shell
npm install @tauri-apps/api
```

## Vue

项目结构
- public: 不会被编译的资源文件
- src/assets: 资源文件，会被编译
- src/components: 组件
- src/App.vue: 入口文件

### SFC 单文件组件

一个文件就是一个组件，一般分层几个部分：
- script: setup只能有一个，其他的script可以有多个
- template: 模板，只能有一个
- style: css

### vue 模板语法

在`<script setup>`里面定义一个变量v，然后就可以在`<template>`里面使用`{{ v }}`用这个变量了。这是最简单的vue的模板语法：

```vue
<template>
<div>
    {{ v }}
</div>
</template>

<script setup lang="ts">
    const v:number = 1;
</script>
```

### vue 指令

vue提供的语法糖指令。比如，v-text：直接显示text。使用方式：

```vue
<template>
<div v-text="v">
</div>
</template>

<script setup lang="ts">
    const v:string = "str";
</script>
```

每种vue指令，都有其用法。

#### v-on
在某事件发生的时候，触发

```vue
<button v-on:click='callback'>
```
v-on有一个简写，使用`@`代替v-on：

```vue
<button @click='callback'>
```

#### v-bind

绑定属性，简写是`:`

```vue
<div :id="my_id">
```

#### v-model

一般是绑定表单元素，比如输入、用户选择等

```vue
<input v-model="a" type="text">
```

注意，这种变量需要用 `ref()` 声明，才能让对应的变量变成响应式的，否则变量不会自动更新

