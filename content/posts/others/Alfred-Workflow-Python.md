---
title: "Alfred Workflow Python Development"
author: "Haobo Gu"
tags: [alfred]
date: 2019-08-20T11:33:02+08:00
---

Learning Alfred workflow python sdk. 

<!--more-->

Install Alfred workflow python sdk first: https://www.deanishe.net/alfred-workflow/installation.html

## Add script runner

There are two components in Alfred which can execute a python script: **script filter** and **run script**. There are a few differences between script filter and run script, we'll talk about it later. 

The usage of both components is similar: ![image-20190820140905988](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-08-20-060906.png)

Note that the Alfred workflow python sdk supports only Python2, since the built-in Python interpreter in Mac OS is Python2.

## Your first python script

The following is a simple example of Alfred Python script:

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-

from __future__ import print_function
import sys
import os


def main(wf):
  	# If the input {query} is not null
    if len(wf.args):
      	# pass {query} to downstream
     		print(wf.args)

if __name__ == '__main__':
    wf = Workflow3()
    sys.exit(wf.run(main))
```

This script passes the input `{query}` to downstream. In Alfred, all contents which is printed to stdout will be passed to downstream as `{query}`.

## Variables

If we want to pass more information to downstream, Alfred variable is what we need.

### Set variables

There are two types of variables: general variable and environment variable. Alfred Python sdk provides difference methods to set different variables. What's more, variable setting is different in **run script** component and **script filter** component. 

##### Set environment variable

Alfred Python provides `set_config` in `workflow.util` to set environment variables. Here is the [API doc](https://www.deanishe.net/alfred-workflow/api/index.html?highlight=set_config#workflow.util.set_config), which is quite easy to understand and use.

```python
from workflow.util import set_config, unset_config

# Set environment variable
set_config(var_name, var_value, bundleid, exportable=False)

# Unset environment variable
unset_config(var_name, bundleid)
```

##### Set variables in script filter component

In the upstream script filter, `root variable` and `item variable` can be used to pass variables to downstream.

```python
# Add item variable
it = wf.add_item(title, subtitle, arg, valid)
it.setvar(var_name, var_value)

# Add root variable
wf.setvar(var_name, var_value)
```

`wf.add_item` adds an item, which is a collection of variables as well as one candidate in alfred window. When the candidate is chosen, the item is ***actioned***. 

When an item is actioned, all variables in `arg` is passed as `{query}` to downstream

`title` and `subtitle` is the text appears in the alfred window.

##### Set variables in run script component

In run script component, `Variables` should be used to store and pass variables.

Here is an example, we initialize a variable called `fail`, and pass it to downstream:

```python
from workflow import Variables
if not success:
  v = Variables()
  v["fail"] = "Fail to run script"
  print(v)
  return
```

### Get variables

In the downstream script, `os.getenv(var_name)` is used to get variables:

```python
import os
var_value = os.getenv(var_name)
```

In other components of Alfred, we can use `{var:var_name}`.

The following is an example: 

![image-20190820141856773](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-08-20-061857.png)

If the variable called `fail` is empty, the downstream component "open file" will be executed with the input `{query}`, where `{query}` is the file path. 

## Logger

In Alfred Python, we cannot use `print` to debug because the printed content will be passed to downstream as `{query}`. But don't worry, the sdk provdes a logger `wf.logger`

Logger initialization: 

```python
from workflow import Workflow3

log = None

if __name__ == '__main__':
    wf = Workflow3()
    log = wf.logger
    sys.exit(wf.run(main))
```

Then we can use `log.error()`, `log.warn()`, `log.debug()`, `log.info()` to debug. All info will be printed in debug window when the script is running:

![image-20190819113138527](http://haobo-markdown.oss-cn-zhangjiakou.aliyuncs.com/markdown/2019-08-20-064331.png)