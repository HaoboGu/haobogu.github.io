---
title: "Learning Rust - 3"
author: "Haobo Gu"
tags: [rust]
date: 2021-07-08T11:29:31+08:00
summary: Traits and generics in Rust
draft: false
---

# Learning Rust - 3

## 泛型

泛型是非常常见的语言特性。使用泛型类型时，可以指定所需操作，而不必考虑定义类型持有的内部类型。在Rust中，使用尖括号`<>`来声明一个结构体的泛型类型：

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let boolean = Point { x: true, y: false };
    let integer = Point { x: 1, y: 9 };
    let float = Point { x: 1.7, y: 4.3 };
    let string_slice = Point { x: "high", y: "low" };
}
```

上面代码中，`T`就是泛型的类型，在实例化的时候，`T`可以被任何确定的类型代替。在上面的定义中，实际上就是声明了`x`和`y`属于同一种数据类型，但是具体类型未指定。如果想把`x`和`y`声明为不同的类型，可以使用：

```rust
struct Point<T, U> {
    x: T,
    y: U,
}
```

## 特性

特性（trait）指一组可以被各种数据类型实现的通用接口，它定义了一组通用的行为。比如我们之前遇到的`Copy`特性，就是代表着一个类型是默认被复制而不是被移动的。如果你想要自己定义的类型默认被复制，那么只需要实现`Copy`特性即可。这个定义和golang中的接口非常相似，区别在于golang中的接口是隐式实现的，而在Rust中我们必须显式声明某个类型实现了某个特性。

我们也可以使用`trait`关键字自己去定义特性：

```rust
trait Area {
    fn area(&self) -> f64;
}
```

上面的代码就定义了”有面积“的这么一个特性，中间定义的函数就代表着，如果一个类型有该特性，那么需要实现计算其面积的`area`方法。比如我们定义一个矩形的结构：

```rust
struct Rectangle {
    width: f64,
    height: f64,
}
```

我们就可以为`Rectangle`结构实现`Area`特性，来计算其面积：

```rust
impl Area for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
    }
}
```

为实现某种类型的特性，我们使用关键字 `impl Trait for Type`，其中 `Trait` 是要实现的特征的名称，`Type` 是实现器结构或枚举的名称。

实现了某种特性之后，我们就可以按照常规方法调用的方式来使用特性里面定义的方法：

```rust
let rectangle = Rectangle {
    width: 10.0,
    height: 20.0,
};

println!("Rectangle area: {}", rectangle.area());
```

## 派生

还有一种情况，比如我定义了下面的类型：

```rust
struct Point {
    x: i32,
    y: i32,
}
```

`Point`中的所有成员都已经实现了`Copy`特性，但是`Point`本身没有实现`Copy`特性，所以默认情况下，`Point`还是会被移动。那么这种情况下如何快速实现`Point`的`Copy`特性呢？Rust提供了`derive`注解，可以为类型快速地自动生成新特性：

```rust
#[derive(Copy)]
struct Point {
    x: i32,
    y: i32,
}
```

如果一个结构的每一个字段都已经实现了某个特性，那么使用`#[derive(trait)]`就能够自动地为某个结构实现该特性。

## 使用特性

有了特性这么一个工具，我们就可以要求某些函数的参数实现特定的特性。换句话说，只要实现某特性的结构都可以作为该函数的参数。比如我们有如下特性，表明可以被转换为json字符串：

```rust
trait AsJson {
    fn as_json(&self) -> String;
}
```

然后我们就可以定义一个函数，这个函数可以把任意可以转换为json的类型打印出来：

```rust
fn print_data_as_json(value: &impl AsJson) {
    println!("{}", value.as_json());
}
```

可以看到，我们声明某个特性为函数参数类型的时候，前面需要加`impl`，意思就是这里实际上是这个特性的具体实现者。

当然，通过泛型我们也可以做到这一点：

```rust
fn print_data_as_json<T: AsJson>(value: &T) { ... }
```

这里就规定了一个泛型类型，这个泛型类型需要实现`AsJson`特性才能被接收。上面的两种实现是等价的。

### 实例：实现一个迭代器

`Iterator`是一个Rust自带的特性，定义如下：

```rust
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

下面是一些解释：

> `Iterator` 具有方法 `next`，调用时它将返回 `Option<Item>`。 只要有元素，`next` 方法就会返回 `Some(Item)`。 用尽所有元素后，它将返回 `None` 以指示迭代已完成。
>
> 此定义使用一些新语法：`type Item` 和 `Self::Item`，它们使用此特征定义关联的类型。 此定义意味着 `Iterator` 特征的每一次实现还需要定义关联的 `Item` 类型，该类型用作 `next` 方法的返回类型。 换句话说，`Item` 类型将是从 `for` 循环块内的迭代器返回的类型。

假设我们需要实现一个`Counter`，用作迭代器且进行计数，首先下面是struct的声明：

```rust
#[derive(Debug)] // 继承Rust自带的Debug特性
struct Counter {
    length: usize,
    count: usize,
}

impl Counter {
  // Counter有一个new函数，用于初始化一个新的Counter
    fn new(length: usize) -> Counter {
        Counter {
            count: 0,
            length,
        }
    }
}
```

然后，我们就实现`Counter`结构的`Iterator`特征。此处需要使用`impl Trait for Struct`格式：

```rust
impl Iterator for Counter {
    type Item = usize; // 我们将通过 usize 进行计数，因此，我们声明相关 Item 类型应为该类型。

    // next方法是实现迭代器的必须方法
    fn next(&mut self) -> Option<Self::Item> {
    
        self.count += 1;
        if self.count <= self.length {
            Some(self.count)
        } else {
            None
        }
    }
}
```

到这里我们就为`Counter`实现了`Iterator`特征。还记得我们可以对一个`Iterator`使用`for item in Iterator`的语法吗？这里我们对`Counter`也可以这样用了：

```rust
fn main() {
    for number in Counter::new(10) {
        println!("{}", number);
    }
}
```



