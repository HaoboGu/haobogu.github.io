---
title: "Alfred Workflow Basics"
author: "Haobo Gu"
tags: [alfred]
date: 2019-08-19T14:25:50.275944+08:00
---

<!--more-->

Before we start to develop an Alfred workflow, we should learn some basic concepts first. 

Prerequisites: https://www.alfredapp.com/help/workflows/

## Workflow development basics

There are five categories of workflow node in Alfred: Triggers, Inputs, Actions, Utilities, Outputs

The two most important categories are **Inputs** and **Actions**. The first one read inputs from various sources, and the latter one are what does the workflow do.

## Inputs

There are 5 types of inputs:

- Keyword

- File filter

  Find a file, pass `file_path` as the {query}

- Dictionary Filter

- List filter

- Script Filter

### File Filter

File types in file filter is used to narrow the scope of searched files. For example, a png file's file types tree (from precise classification to broadest classification) can be:

public.png -> public.image -> public.data -> public.item -> public.content

### Script Filter

The most used input filter. A bash script is used to process the input, including execute a python/ruby/perl/go/... program. Hence, in most cases, we can do all what we want to do in script filter. 

## Actions

There are 12 actions in Alfred, including open file/app/url, run system command, terminal command, etc. If the given actions cannot satisfy your requirement, Alfred also provide an action called run script. Just like a script filter. But there are a few differences between script filter and run script. We'll talk about it latter.

## Variables

Please read https://www.alfredapp.com/help/workflows/advanced/variables/ as the prerequisites first.

Variables are just like variables in programming, which stores values, results and informations. 

In Alfred, we can use `{var:var_name}` to get the value of variable `var_name`. BTW, `{query}` can be also regarded as a special variable. 

A general variable is valid for node in downstream, and the environment variable is always valid in the workflow. 

#### Placeholder

Alfred offers dynamic placeholders, which allow you to insert dynamically-created content. All variables can be accessed using placeholder. The placeholder's format is `{placeholder:variation}`, where `variation` is used to specify the advanced options when Alfred replaces the placeholder with the dynamic content. 

There are some useful placeholders, all of them have variations, for those details, pls refer to [this page](https://www.alfredapp.com/help/workflows/advanced/placeholders)

- `{query}`
- `{var:var_name}`, `{allvars}`
- `{date}`, `{time}`, `{datetime}`
- `{clipboard:0}`, `{clipboard:n}`: n-th item in clipboard 
- `{random}`

What's more, placeholders have modifiers, which provides more possibilities. The modifiers follow this format: `{placeholder:variation.modifier}`

The following are allowed modifiers:

- `uppercase`, `lowercase`, `captitals`, `capitalcase`

- `trim`

- `reverse`

- `stripdiacritics`: remove accented characters

- `stripnonalphanumeric`: remove non-alphanumeric characters

  

## Debugger

Debugger is quite useful in the workflow development. It prints values you want to debug window. By default, it prints `{query}` and `{allvars}`. 

![image-20190819113138527](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-08-19-033139.png)



