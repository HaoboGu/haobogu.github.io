---
title: "Learn Makefile - part 1"
author: "Haobo Gu"
tags: [makefile, c, c++]
date: 2022-09-26T23:06:12+08:00
summary: Learn Makefile - part 1
---

# Makefile

## 程序的编译和链接

首先需要了解一下程序的编译和链接。编译就是把源代码编译成目标（obj）文件，而链接是把所有的目标文件（包括函数、全局变量等）链接生成可执行文件。

## Makefile介绍

使用make命令就需要Makefile文件。Makefile文件会告诉make命令如何编译、链接程序。

### Makefile规则

首先看一下Makefile中最基础的规则：

```
目标：依赖
  命令
```

规则很简单：当**依赖**更新时，执行**命令**生成**目标**。这个简单的规则就是Makefile最核心的内容。下面稍微介绍一下这三个概念：

- 目标：目标文件，可以是编译产物（obj）、链接产物（可执行文件），还可以是**标签**（Label，后面会解释）
- 依赖：生成**目标**所依赖的文件或者其他**目标**
- 命令：一般是shell命令，用于从依赖生成目标。make会比较目标和依赖的生成时间，如果依赖比较新，那么就会执行命令

下面是一个例子：

```makefile
# 最终目标文件--可执行文件
edit : main.o kbd.o
	cc -o edit main.o kbd.o

# 中间目标文件
main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c

# 伪目标文件
clean :
	rm edit main.o kbd.o
```

可以看到里面定义的3个目标，第一个是生成`edit`可执行文件，第二个是编译中间文件，第三个是`clean`。需要提到的是，这里目标`clean`并不是一个文件，而是一个label。可以看到这个目标没有任何依赖，那就说明只要运行`make clean`，下面的命令一定会被执行。

### make是如何工作的

在默认情况下， 直接执行`make`那么：

1. make会去找`Makefile`
2. `Makefile`中，找到**第一个目标**，作为终极目标
3. 然后嵌套地去根据终极目标的所有依赖去找其他目标，然后根据需要执行命令

需要注意的是，make不会管**命令**的错误，它只管文件的依赖性，在依赖找不到的情况下直接报错退出，而命令报错则不会。

### Makefile中的变量

和C语言的宏类似，在Makefile中可以定义变量，以提升Makefile的可读性和可维护性。变量的定义和使用很简单：

```makefile
# 定义objects变量
objects = main.o kbd.o
# 使用objects变量
edit : $(objects)
```

## make的自动推导

`make`有自动推导的能力，即它会记住前面的命令，这样就不用每次再写同样的命令了。
另外，`make`还有一个功能就是，每当`make`看到一个`.o文件`，它都会把对应的`.c文件`加入到依赖里面。
使用这两个特性，我们的Makefile就变成了这样：

```makefile
# 变量定义
objects = main.o kbd.o

# 最终目标文件
edit : $(objects)
		cc -o edit $(objects)

# 中间目标文件
main.o : defs.h
kbd.o : defs.h command.h

# 伪目标文件
.PHONY : clean
clean :
		rm edit $(objects)
```

## 清空目标文件的规则

每个Makefile中都应该写一个清空目标文件（.o和执行文件）的规则，这不仅便于重编译，也很利于保持文件的清洁。还记的之前例子里面的`clean`目标吗？没错就是这个。不过更好的写法是：

```makefile
.PHONY : clean
clean :
		-rm edit $(objects)
```

这里，`.PHONY`表示`clean`是一个**伪目标**，而`rm`前面的`-`表示也许某些文件出现问题，但不要管，继续做后面的事。
另外，还有一个不成文的规矩是，`clean`一般都放在Makefile的**最后**（相对应地，第一个目标是默认目标或者说终极目标）。

## Makefile总述

上面介绍了关于`make`和Makefile的基础知识，下面我们就深入了解Makefile。

### 选择执行的Makefile

默认情况下， `make`会使用当前目录下的`Makefile`或者`makefile`。当然你也可以手动地指定Makefile：

```shell
make -f your_make_file
make --file your_make_file
```

### 引用其他的Makefile

在一个Makefile中，你也可以引用其它的Makefile，只需要使用`include`关键字即可：

```makefile
# 引入其他makefile
include foo.make *.mk $(many_makefiles)
# 如果文件不存在，也继续
-include not_exist.make
```

被引入的Makefile会原样地放在这里（当然，也是递归地搜索所有依赖的Makefile）。如果在`include`前面有一个`-`，则说明即使有问题也继续（和上面命令里面的含义一样）。

### Makefile的书写规则

在Makefile里，规则的顺序是非常重要的，这是因为Makefile中只有一个**终极目标**。一般来说，第一个规则里面的目标就是终极目标，如果第一个规则里有很多目标，那其中第一个目标是终极目标。整个`make`命令所要完成的也是这个目标。
在上面介绍中，我们已经介绍了规则的写法。实际上，规则还有一种写法：

```makefile
targets : prerequisites ; command
```

目标和依赖都可以是多个文件，以空格分隔。而命令如果和目标/依赖在同一行，那么必须分号隔开；如果不在同一行，那么新行必须以**TAB**开头：

```makefile
targets : prerequisites
     command
# ↑ 注意这里是tab
```

### 通配符

Makefile支持3种通配符（wildcard）：`*`, `?`,`[...]`。定义和linux里面一样，不再细说了。

### 伪目标

在上面的代码中，我们提到了`.PHONY`定义的是一个伪目标。为啥叫伪目标呢，这是因为实际上`clean`那个命令并不是一个目标**文件**，而只是一个标签。执行`make clean`也不生成任何目标文件，所以叫伪目标。
伪目标在Makefile中很有用，比如如果你想使用`make`一下子生成好多文件，那么就可以这样写：

```makefile
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
		cc -o prog1 prog1.o utils.o

prog2 : prog2.o
		cc -o prog2 prog2.o

prog3 : prog3.o sort.o utils.o
		cc -o prog3 prog3.o sort.o utils.o
```

这里，我们的终极目标`all`实际上就是一个伪目标，然后它依赖了三个目标。这样，在执行`make all`的时候，下面三个目标会总是被执行。

### 静态模式规则

在很多时候，我们的目标和依赖都是很相似的，如下面的规则：

```makefile
a.o : a.c
    $(CC) -c $(CFLAGS) a.c -o a.o
b.o : b.c
    $(CC) -c $(CFLAGS) b.c -o b.o
```

这个时候，我们就可以使用模式规则：

```makefile
targets: target_pattern : prereq_pattern
    commands
```

这里，目标还是保持不变，但是后面的依赖变为了由模式生成。一般最常用的就是`%.o: %.c`，即把target中所有的`.o`文件都替换成`.c`文件：

```makefile
objects = foo.o bar.o
$(objects): %.o: %.c
```

结合Makefile中的`filter`函数，就可以批量生成成千上万的`.o`文件了：

```makefile
files = foo.elc bar.o lose.o
$(filter %.o,$(files)): %.o: %.c
		$(CC) -c $(CFLAGS) $< -o $@
```

这里，又有了三个新知识点：`filter`函数、`$<`和`$@`。`filter`是make自带的函数，而的和`$<`和`$@`则是自动化变量，`$<`表示所有的依赖目标集（也就是`foo.c`、`bar.c`），`$@`表示目标集（也就是`foo.o`、`bar.o`）。关于函数和自动化变量，我们后面还会详细说明。