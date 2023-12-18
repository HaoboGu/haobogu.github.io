---
title: "Effective Go"
author: "Haobo Gu"
tags: [go]
date: 2020-04-10T17:28:08.734127+08:00
draft: false
summary: Learn effective go
---
## 命名

变量/常量命名：驼峰

接口名：XXXer

Getter/Setter：对于`obj.field`，getter用`obj.Field()`，setter用`obj.SetField()`

## 控制结构

### if

可以在if中定义变量：

```go
if ok:=isOk();ok {
  // OK
}

var v string
var err error
if v, err = doSomething(); err != nil {
  // 处理错误
}
// 继续
```

### for/switch

for有三种情况

```go
// 如同C的for循环
for init; condition; post { }

// 如同C的while循环
for condition { }

// 如同C的for(;;)循环
for { }
```

switch可以替代多个if-else

```go
func unhex(c byte) byte {
	switch {
	case '0' <= c && c <= '9':
		return c - '0'
	case 'a' <= c && c <= 'f':
		return c - 'a' + 10
	case 'A' <= c && c <= 'F':
		return c - 'A' + 10
	}
	return 0
}
```

一个case可以有多个条件

```go
switch c {
	case ' ', '?', '&', '=', '#', '+', '%':
		return true
	}
```

类型判断的典型用法

```go
t = functionOfSomeType()
switch t := t.(type) {
default:
	fmt.Printf("unexpected type %T", t)       // %T 输出 t 是什么类型
case bool:
	fmt.Printf("boolean %t\n", t)             // t 是 bool 类型
case int:
	fmt.Printf("integer %d\n", t)             // t 是 int 类型
case *bool:
	fmt.Printf("pointer to boolean %t\n", *t) // t 是 *bool 类型
case *int:
	fmt.Printf("pointer to integer %d\n", *t) // t 是 *int 类型
}
```

## 函数

返回的结果变量可以先命名

```go
func nextInt(b []byte, pos int) (value int, nextPos int) {
  // 直接用
	nextPos = 1
  // 不带参数的返回，默认返回当前值，即0和1
  return 
}
```

### defer

`defer function(param)`可以把`function`推迟到当前函数结束的时候执行。需要注意的是，如果`param`也是一个函数，那么`param`这个函数会先执行

```go
func function(p int) {
  // notDeferred(p)会先被执行
  // function推出前，deferred()才会被执行
  defer deferred(notDeferred(p))
}
```

## 切片Slice

如果把Slice当做参数传入某函数，那么函数对slice的修改，对上层是可见的。即，传进去的类似一个引用，而不是形参。

```go
// 典型例子，Read结果会被存在buf中
func (file *File) Read(buf []byte) (n int, err error)
```

**但是**：切片自身（其运行时数据结构包含指针、长度和容量）是通过值传递的。参考`mySlice = append(mySlice, item)`

## Map

如果通过不存在的键来取值，会返回**对应类型的零值**。

如果需要区分是不存在，还是存的值本来就是零值，可以使用逗号OK法：

```go
if re, ok := myMap[key]; ok {
  // 存在值
}
```

Go中没有集合，一般是使用`map[string]bool`作为集合类型

## 指针

```go
type ByteSlice []byte
// 必须返回[]byte重新对slice赋值，即s = s.Append(data)
func (slice ByteSlice) Append(data []byte) []byte {
}
// 区别在于，这个Append可以不用赋值，直接可以修改ByteSlice：s.Append(data)
func (p *ByteSlice) Append(data []byte) {
}
```

在用的时候，其实都是`s.Append()`，这是因为使用指针作为接收者的声明形式会被自动识别，在调用的时候会被自动改成`(&s).Append()`

## 接口

Go里面的接口是松散的，只要实现了一个接口里面定义的方法，就相当于实现了这个接口。不需要显式地声明XXX实现了某interface

```go
type MyInterface {
  Do()
}

type MyType struct {}

// 实现了Do()，这里就隐式实现了MyInterface接口
func (p *MyType) Do() {
  
}

// Do2()可能是其他接口里面的函数
func (p *MyType) Do2() {
  
}
```

## 并发

在go里面，多个独立的线程从来不会主动共享：

> Do not communicate by sharing memory; instead, share memory by communicating.
>
> 不要通过共享内存来通信，而应通过通信来共享内存。

### Goroutines

goroutine是并发的、轻量级的，使用也很简单：`go`

```go
go list.Sort() // 并发运行 list.Sort，无需等它结束。
```

### Channels

Channel就是上面提到的，goroutine间通信的方式。Channel需要使用make来分配内存：

```go
ci := make(chan int)     // 整数类型的无缓冲信道
cs := make(chan *os.File, 100)  // 指向文件指针的带缓冲信道
```

这里，缓冲是用来阻塞**发送者**的，如果没有缓冲，那么在接收者接收到值之前，发送者就一直阻塞；如果有缓冲，那么发送者仅在值被复制到缓冲区前阻塞。

接收者在接收到相应的值之前会一直阻塞。以上面的Sort为例：

```go
c := make(chan bool)  // 分配一个信道
// 在Go程中启动排序。当它完成后，在信道上发送信号。
go func() {
	list.Sort()
	c <- true  // 发送信号，什么值无所谓。
  fmt.Print("sorted")  // 接收者接收到信号之前，不会print
}()
doSomethingForAWhile()
success <-c   // 等待排序结束，接收到true
```



