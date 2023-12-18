---
title: "Learning Rust - 1"
author: "Haobo Gu"
tags: [rust]
date: 2021-07-07T10:28:08+08:00
draft: false
summary: Rust language basics
---

# Learning Rust - 1

学习[微软Rust教程](https://docs.microsoft.com/zh-cn/learn/paths/rust-first-steps/)的笔记。

## Rust的特点

Rust 是现有系统软件语言（如 C 和 C++）的一种安全替代语言。 与 C 和 C++ 一样，Rust 没有大型运行时或垃圾回收器，这几乎与所有其他现代语言形成了鲜明对比。 但是，与 C 和 C++ 不同的是，Rust 保证了内存安全。 Rust 可以避免很多与在 C 和 C++ 中遇到的内存使用错误相关的 bug。

Rust 有以下优点，非常适合各种应用程序：

- 类型安全：编译器可确保不会将任何操作应用于错误类型的变量。
- 内存安全：Rust 指针（称为“引用”）始终引用有效的内存。
- 无数据争用：Rust 的 borrow 检查器通过确保程序的多个部分不能同时更改同一值来保证线程安全。
- 零成本抽象：Rust 允许使用高级别概念，例如迭代、接口和函数编程，将性能成本控制在最低，甚至不会产生成本。 这些抽象的性能与手工编写的底层代码一样出色。
- 最小运行时：Rust 具有非常小的可选运行时。 为了有效地管理内存，此语言也不具有垃圾回收器。 在这一点上，Rust 非常类似于 C 和 C++ 之类的语言。
- 面向裸机：Rust 可以用于嵌入式和“裸机”编程，因此适合用于编写操作系统内核或设备驱动程序。

## 了解Cargo

Cargo是Rust语言的生成工具和依赖管理器，Cargo为管理Rust程序带来了很多便利。

Cargo 可以为你做许多事情，包括：

- 使用 `cargo new` 命令创建新的项目模板。
- 使用 `cargo build` 编译项目。
- 使用 `cargo run` 命令编译并运行项目。
- 使用 `cargo test` 命令测试项目。
- 使用 `cargo check` 命令检查项目类型。
- 使用 `cargo doc` 命令编译项目的文档。
- 使用 `cargo publish` 命令将库发布到 crates.io。

### 尝试Cargo

首先使用Cargo创建一个新的Rust工程：

```shell
cargo init <project-name>
```

创建完工程之后，使用VSCode打开刚才创建的文件夹，可以看到cargo已经自动为你生成了`cargo.toml`和`src/main.rs`两个文件。

- Cargo.toml 是 Rust 代码库的配置文件，用于管理依赖版本等。如果你有其他语言的经验，可以类比`package.json`或者`go.mod`等
- src 子目录中的 main.rs 文件为当前工程的主入口文件，里面的`fn main()`即为主入口函数

`cargo init`生成的工程是一个可运行的Rust的HelloWorld工程。下面还有一些命令可以尝试：

运行当前工程：

```shell
cargo run
```

编译当前工程：

```shell
cargo build
```

编译当前工程（发布使用）：

```shell
cargo build --release
```

编译完成之后，你可以在`target/debug`和`target/release`目录下看到编译出的可执行文件。

## 语言基础

### 变量

#### 声明变量

使用关键字`let`

```rust
// Declare a variable
let a_number;
    
// Declare a second variable and bind the value
let a_word = "Ten";
    
// Bind a value to the first variable
a_number = 10;

println!("The number is {}.", a_number);
println!("The word is {}.", a_word);
```

#### 变量的可变性

和一般的语言不一样，默认情况下，Rust的变量是**不可变的**：

```rust
let a_number = 10;
a_number = 15; // 此行报错！
```

如果想要变量可变，那么需要使用关键字`mut`:

```rust
let mut a_number = 10;
a_number = 15; // 可以
```

为什么要默认变量是不可变的呢？《The Rust Programming Language》有如下解释：

> 如果一部分代码假设一个值永远也不会改变，而另一部分代码改变了这个值，第一部分代码就有可能以不可预料的方式运行。不得不承认这种 bug 的起因难以跟踪，尤其是第二部分代码只是 **有时** 会改变值。
>
> Rust 编译器保证，如果声明一个值不会变，它就真的不会变。这意味着当阅读和编写代码时，不需要追踪一个值如何和在哪可能会被改变，从而使得代码易于推导。
>
> 不过可变性也是非常有用的。变量只是默认不可变；正如在第二章所做的那样，你可以在变量名之前加 `mut` 来使其可变。除了允许改变值之外，`mut` 向读者表明了其他代码将会改变这个变量值的意图。
>
> -- [变量与可变性 - Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/ch03-01-variables-and-mutability.html)

##### 不可变变量和常量的区别：

[变量与可变性 - Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/ch03-01-variables-and-mutability.html#变量和常量的区别)

总结一下：

1. 不允许对常量使用 `mut`。常量不光默认不能变，它总是不能变。
2. 声明常量使用 `const` 关键字而不是 `let`，并且 *必须* 注明值的类型
3. 常量只能被设置为常量表达式，而不能是函数调用的结果，或任何其他只能在运行时计算出的值
4. 常量可以在任何作用域中声明，包括全局作用域

#### 变量隐藏

可以重复使用let声明同名的变量，这样的话变量名会被绑定在新的值上面，旧的变量就被“隐藏”了。需要注意的是，**旧变量仍然存在**。

```rust
let a_number = 10;let a_number = 15; // 隐藏上面的变量，但是上面的变量不会被删除，仍存在于内存中println!("{}", a_number); // 15
```

### 数据类型

Rust是静态类型语言。在声明变量时，编译器会自动推断变量类型，但是也可以使用`:`（类似typescript）手动指定变量类型：

```rust
let number1 = 15; // 默认是i32类型let number2: i64 = 15; // 手动声明i64整型
```

#### 数字类型

分为整数、浮点数。具体表见：[数据类型 - Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/ch03-02-data-types.html#整型)

默认的整型和浮点数类型为：`i32`和`f64`

#### 布尔类型

`true` or `false`

#### 字符

字符类型为`char`，使用单引号括住：

```rust
let s = 's';let emoji = '😃';
```

#### 字符串

Rust中，有好几种字符串类型：`&str`（字符串引用）, `String`（堆上字符串）等。具体使用还是有区别的。可以参考：[字符串 - Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/ch08-02-strings.html)。现在，可以先简单地认为 `String` 是可随程序运行而更改的文本数据。 `&str` 引用是文本数据的不可变视图，不会随着程序运行而改变。

#### 元组tuple

元组是固定长度的分组，使用`(<value1>, <value2>, ...)`表示。每个value的类型可以不一样。获取元组中的元素，可使用`tuple.index`

```rust
// Declare a tuple of three elementslet tuple_e: (char, i32, bool) = ('E', 5i32, true);// Use tuple indexing and show the values of the elements in the tupleprintln!("Is '{}' the {}th letter of the alphabet? {}", tuple_e.0, tuple_e.1, tuple_e.2);
```

### 控制流

#### if else

语法很简单：

```rust
if condition {    } else if another_condition {    } else {    }
```

用的时候，`ifelse`块还可以充当表达式：

```rust
let formal = true;let greeting = if formal { // if used here as an expression    "Good day to you."     // 注意，此处没有分号结尾，即 "Good day to you." 为这个if块的返回值} else {    "Hey!"                 // 返回 “Hey!"};println!("{}", greeting)   // prints "Good day to you."
```

### 复杂数据结构

#### 结构体

使用关键字`struct`定义，结构类型名称采用大写形式。

> Rust 支持三种结构类型：经典结构、元组结构和单元结构。 这些结构类型支持使用各种方式对数据进行分组和处理。
>
> - “经典 [C 结构](https://wikipedia.org/wiki/Struct_(C_programming_language))”最为常用。 结构中的每个字段都具有名称和数据类型。 定义经典结构后，可以使用语法 `<struct>.<field>` 访问结构中的字段。
> - 元组结构类似于经典结构，但字段没有名称。 要访问元组结构中的字段，请使用索引元组时所用的语法：`<tuple>.<index>`。 与元组一样，元组结构中的索引值从 0 开始。
> - “单元结构”最常用作标记。 我们将在了解 Rust 的特征功能时，将深入了解单元结构之所以实用的原因。
>
> 以下代码显示三种结构类型变体的示例定义：
>
> ```rust
> // Classic struct with named fields
> struct Student { name: String, level: u8, pass: bool }
> 
> // Tuple struct with data types only
> struct Grades(char, char, char, char, f32);
> 
> // Unit struct
> struct Unit;
> ```

结构体实例化：

```rust
// Instantiate classic struct, specify fields in random order, or in specified order
let user_1 = Student { name: String::from("Constance Sharma"), remote: true, level: 2 };
let user_2 = Student { name: String::from("Dyson Tan"), level: 5, remote: false };

// Instantiate tuple structs, pass values in same order as types defined
let mark_1 = Grades('A', 'A', 'B', 'A', 3.75);
let mark_2 = Grades('B', 'A', 'A', 'C', 3.25);
```

当然，我们也可以为结构体定义成员函数，使用`impl`关键字即可

```rust
// Classic struct with named fields
struct Student { name: String, level: u8, pass: bool }
impl Student {
    fn get_name(&self) -> &String {
        return &self.name;
    }
}

fn main() {
    let s = Student{name: "n".to_string(), level:1, pass:true};
    println!("{}", s.get_name());
}
```

### 枚举

关键字`enum`。需要注意的是，Rust的枚举中，每个值可以有不同的类型。这样的话，在使用某个枚举类型时，必须接受其下面所有值的类型：

```rust
enum WebEvent {
    // An enum variant can be like a unit struct without fields or data types
    WELoad,
    // An enum variant can be like a tuple struct with data types but no named fields
    WEKeys(String, char),
    // An enum variant can be like a classic struct with named fields and their data types
    WEClick { x: i64, y: i64 }
}
```

一般来说，我们不直接在枚举里面定义一个复杂的结构，而是在外面定义好相应的结构体之后，在枚举里面使用：

```rust
// Define a tuple struct
struct KeyPress(String, char);

// Define a classic struct
struct MouseClick { x: i64, y: i64 }

// Redefine the enum variants to use the data from the new structs
// Update the page Load variant to have the boolean type
enum WebEvent { WELoad(bool), WEClick(MouseClick), WEKeys(KeyPress) }
```

在使用枚举时，采用运算符`::`来指定具体的枚举值：

```rust
// bool
let we_load = WebEvent::WELoad(true);

// Instantiate a MouseClick struct and bind the coordinate values
let click = MouseClick { x: 100, y: 250 };
// Set the WEClick variant to use the data in the click struct
let we_click = WebEvent::WEClick(click);
```



### Rust函数

Rust的函数使用关键字`fn`声明：

```rust
fn main() {
    println!("Hello, world!");
}
```

函数的返回值由`->`确定，参数填在`()`里面，使用`:`指定类型：

```rust
fn is_divisible_by(dividend: u32, divisor: u32) -> bool {
  ...
}
```

在函数体中，**大多数**的语句是分号`;`结尾的。如果不是分号结尾的语句，则有可能是函数的**返回值**！

```rust
fn getFive() -> i32 {
    5
}

// 等同于
fn getFive() -> i32 {
    return 5;
}
```



###集合类型

Rust中自带了一些常见的集合类型：数组、向量、HashMap等

#### 数组

Rust中的数组是具有**相同数据类型**和**固定长度**的对象集合。定义和索引：

```rust
// Declare array, initialize all values, compiler infers length = 7
let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
// Declare array, first value = "0", length = 5
let bytes = [0; 5];

// Set first day of week
let first  = days[0];
```

#### 向量

Rust中的向量是**长度可变**的**相同数据类型**的对象集合。向量声明：

```rust
// Declare vector, first value = "0", length = 5
let zeroes = vec![0; 5];
// Create empty vector, declare vector mutable so it can grow and shrink
let mut fruit = Vec::new();
```

注意，代码中的`vec!`是一个**宏**，而`Vec::new()`为调用`Vec`中的`new()`方法

索引、添加和删除值：

```rust
fruit.push("Apple");
fruit.push("Banana");
fruit.push("Cherry");
let cherry = fruit.pop();
let apple = fruit[0];
let banana = fruit[-1];
```

#### HashMap

Rust中的HashMap定义在标准库中，因此在使用前需要使用

```rust
use std::collections::HashMap;
```

引入。`use`关键字和其他语言中的`import`类似，用于导入。

初始化，添加、获取、删除元素：

```rust
let mut reviews: HashMap<String, String> = HashMap::new();

reviews.insert("Ancient Roman History".to_string(), "Very accurate.".to_string());
reviews.insert("Programming in Rust".to_string(), "Great examples.".to_string());

let key: &str = "Programming in Rust";
let v = reviews.get(key);

reviews.remove(key);
```

### 循环

Rust中，提供了三种循环：`loop`, `while`, `for`

#### loop

Rust中的`loop`为无限循环，只能使用`break`跳出。使用`break`时，还能顺带返回一个值：

```rust
let mut counter = 1;
// stop_loop is set when loop stops
let stop_loop = loop {
    counter *= 2;
    if counter > 100 {
        // Stop loop, return counter value
        break counter;
    }
};
// Loop should break when counter = 128
println!("Break the loop at counter = {}.", stop_loop);
```

如果`loop`中有多个`break`，那么每处返回的类型需要一致。

#### while

和其他语言的`while`没什么区别：

```rust
while condition {
  ...
}
```

#### for

对于数组之类的数据结构，可以用`for <value> in <list>`

```rust
let big_birds = ["ostrich", "peacock", "stork"];
for bird in big_birds {
  ...
}
// 也可使用iter()
for bird in big_birds.iter() {
  ...
}
```

另外一种常见的用法是`for idx in a...b`，其中，`a...b`表示从`a`开始，步长为1迭代到`b`（不包含b)：

```rust
for number in 0..5 {
    // 0, 1, 2, 3, 4
}
```

## 错误处理

### panic

panic 是 Rust 中最简单的错误处理机制。发生panic时，Rust会输出一条错误消息、清理资源，然后退出程序。可以调用`panic!`宏来使当前进程panic。一般来说，只有在程序遇到无论如何都恢复不了的错误时使用：

```rust
fn main() {
    panic!("Farewell!");
}
```

### Option

在Rust中，使用`Option<T>`处理可能为空的值。其他语言中，会有`null`, `nil`, `None`之类的值表示空值，在Rust中，除了与其他语言（比如C）交互时，其他情况下基本都不会使用`null`。

`Option<T>`是一个带泛型的枚举：

```rust
enum Option<T> {
    None,     // The value doesn't exist
    Some(T),  // The value exists
}
```

那什么时候会用到Option呢？下面就是一个例子：

> 在前面的单元中，我们提到尝试访问矢量的不存在的索引会导致程序 `panic`，但你可以通过使用 `Vec::get` 方法（该方法返回 `Option` 类型，而不是 panic）来避免这种情况。 如果该值存在于指定的索引处，系统会将其包装在 `Option::Some(value)` 变体中。 如果索引超出界限，则它会改为返回 `Option::None` 值。
>
> ```rust
> let fruits = vec!["banana", "apple", "coconut", "orange", "strawberry"];
> 
> // pick the first item:
> let first = fruits.get(0);
> println!("{:?}", first); // Some("banana")
> 
> // pick the 99th item, which is non-existent:
> let non_existent = fruits.get(99);
> println!("{:?}", non_existent); // None
> ```

Rust提供了多种方法来处理Option值：

1. `match`：类似其他语言的switch，针对Option中的每种情况分别处理

   ```rust
   match Option<T> {
     Some(value) => { ... }
     None => {... }
   }
   
   // 下面是一个实例
   let fruits = vec!["banana", "apple", "coconut", "orange", "strawberry"];
   for &index in [0, 2, 99].iter() {
       // fruits.get(index)返回一个Option<String>
       match fruits.get(index) {
           Some(&"coconut") => println!("Coconuts are awesome!!!"),
           Some(fruit_name) => println!("It's a delicious {}!", fruit_name),
           None => println!("There is no fruit! :("),
       }
   }
   ```

2. `if let`：如果只关心Option中的某一个特定值

   ```rust
   let a_number: Option<u8> = Some(7);
   
   // 如果我只关心这个数字为7的情况，此时适合使用if let
   if let Some(7) = a_number {
       println!("That's my lucky number!");
   }
   ```

3. `unwrap()`/`expect()`：直接获取Option中的Some值，但是Option为None，会直接panic。区别在于`expect()`可以自定义panic的报错信息

   ```rust
   let a: i32 = Some(1).unwrap();
   
   let empty_gift: Option<&str> = None;
   empty_gift.unwrap(); // panic!
   empty_gift.expect("the gift is none!"); // panic with given message!
   //    thread 'main' panicked at 'the gift is none!'
   ```

4. `unwrap_or(<default_value>)`：如果Option为None，则使用默认值

### Result

Rust的`Option`提供了对空值的处理，而对于可能出现的程序的错误，Rust提供了`Result<T, E>`枚举来处理：

```rust
enum Result<T, E> {
    Ok(T):  // A value T was obtained.
    Err(E): // An error of type E was encountered instead.
}
```

`Result`枚举也非常好理解：要么程序运行OK，返回一个T类型的值；要么程序运行Err，返回一个E类型的错误。和`Option`类似，`Result`也提供了`unwrap()`和`expect()`方法直接获取`OK()`包的值。如果返回的是`Err`，则会panic。

能够用于`Option`的`match`和`if let`，也可以用于`Result`

