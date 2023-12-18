---
title: "Learning Rust - 2"
author: "Haobo Gu"
tags: [rust]
date: 2021-07-07T16:55:48+08:00
summary: Ownership, borrowing and lifetime in Rust
draft: false

---

# Learning Rust - 2

## 所有权和借用

所有权（ownership）系统是Rust最为与众不同的特性，它可以让Rust更好地管理内存、保障内存安全而不需要垃圾回收（GC）

### 作用域

在Rust中，作用域使用大括号`{}`表示。一个作用域内定义的变量只在该作用域内有效：

```rust
{
    let mascot = String::from("ferris");
    println!("{}", mascot); // 有效
}
println!("{}", mascot); // 编译错误！mascot只在上面{}内的作用域有效
```

每当超出变量的范围时，变量及其对应的内存会被释放。

### 所有权转移

在定义一个变量的时候，实际上是把一个数据绑定在一个变量名上。如`let a = String::from("str");`，实际上就是把一个整数绑定在变量名`a`上面。因此，在Rust中，变量也被称为“绑定”。绑定有时候会更为贴切一点，因为Rust中的变量，默认是不可变的（还记得`mut`关键字吗）。

对于一个绑定来说，其数据就是由被绑定名“拥有”的，这就是Rust中所有权的简单理解。如上面`let a = String::from("str");;`的例子，就可以简单理解为变量名`a`拥有数据`str`。

那么，看下面的语句：

```rust
let b = a;
```

这个语句就是把a所拥有的数据`1`转移给了变量`b`，这就叫所有权的转移。之后，`a`就不再拥有数据`1`了，下面的代码将会报错：

```rust
let a = String::from("str"); // 把 str 绑定给 a
let b = a; // 把 a 拥有的数据转移给了 b
println("{}", a); // 编译错误！a的值已经被移走了
```

类似的事情也发生在调用函数时：

```rust
fn process(input: String) {}

fn caller() {
    let s = String::from("Hello, world!");
    process(s); // Ownership of the string in `s` moved into `process`
    process(s); // Error! ownership already moved.
}

// 报错信息
    error[E0382]: use of moved value: `s`
     --> src/main.rs:6:13
      |
    4 |     let s = String::from("Hello, world!");
      |         - move occurs because `s` has type `String`, which does not implement the `Copy` trait
    5 |     process(s); // Transfers ownership of `s` to `process`
      |             - value moved here
    6 |     process(s); // Error! ownership already transferred.
      |             ^ value used here after move
```

第二个`process(s)`将会报错。

在 Rust 中，所有权转移（即移动）是默认行为。此模式对编写 Rust 代码的方式有着深远的影响。 它是 Rust 提出的内存安全承诺的核心。

### 复制

那么，如果我想复制一个值而不是移动它呢？

你可能已经注意到上面的报错信息，有提到一个叫`Copy`的特性（trait）。特性可以大概理解为golang里面的接口interface，只要实现了该特性那么就能在相应的地方使用。在这里，如果一个对象实现了Copy特性，那么它就会被复制而不是被移动。例子就是`u32`类型，`u32`类型默认已经实现了`Copy`的特性，因此，下面的代码不会报错：

```rust
fn process(input: u32) {}

fn caller() {
    let s = 1u32;
    process(s); // Ownership of the number in `n` copied into `process`
    process(s); // `n` can be used again because it wasn't moved, it was copied.
}
```

### 显式克隆

还有一种解决方案是，显式地复制一份数据，这样复制出来的数据被移动，旧的值还在。这个时候调用`.clone()`函数即可：

```rust
fn process(s: String) {}

fn main() {
    let s = String::from("Hello, world!");
    process(s.clone()); // Passing another value, cloned from `s`.
    process(s); // s was never moved and so it can still be used.
}
```

但是，每次显式克隆都需要完整复制一份数据，速度会很慢，而且浪费内存

有没有更优雅的办法呢？ 答案肯定是有的，这就涉及到Rust中另外一个重要的概念：借用（borrow）

### 借用

借用的意思也很直接：所有权还是你的，我就借过来用一下。在Rust中，借用是通过**引用**提供的：

```rust
let greeting = String::from("hello");
let greeting_reference = &greeting; // We borrow `greeting` but the string data is still owned by `greeting`
println!("Greeting: {}", greeting); // We can still use `greeting`
```

和C++的引用一样，前面加个`&`即表示是该变量的引用。

> 在上面的代码中，我们使用 `&` 借用了 `greeting`。 `greeting_reference` 的类型为 `&String`。 由于我们只借用了 `greeting`，并没有**移动**所有权，因此，在我们创建 `greeting_reference` 之后仍然可以使用 `greeting`。

既然是借用，那么我们控制使用者如何使用：

- 如果我们不想使用者修改值，那么就传一个**不可变引用**，即`&T`
- 如果允许使用者修改，那么传一个**可变引用**，即`&mut T`

对于一个变量来说，只能有一种类型的引用（可变或不可变）。对于不可变引用来说，多少都行；但是变量的**只能有一个可变引用**。

如何理解呢？假设我把变量借用出去，如果我不允许使用者修改，那么我借给多少人都无所谓，反正不会变；如果我允许使用者修改，那么在同一时刻我只能借给一个人，否则一个变量可能会被多个使用者在不同地方同时修改，会出问题。

这也是其他语言经常会产生bug的地方：某个变量在意想不到的地方被修改了，但是我不知道，还按照没有被修改的情况使用。Rust在很大程度上可以避免这个问题：不想让你改，那么就使用不可变引用。

### 生命周期

使用引用还会出现另外一个问题：引用的项如果被释放了，那么我这个引用就指向了一块无效的内存，这就是悬垂指针。

```rust
fn main() {
    let x;
    {
        let y = 42;
        x = &y; // We store a reference to `y` in `x` but `y` is about to be dropped.
    }
    println!("x: {}", x); // `x` refers to `y` but `y has been dropped!
}
```

C/C++经常会遇到这种问题，但是Rust通过生命周期保证了所有引用都始终引用有效的项。上面的代码编译会报错，因为`y`在其作用域结束时已经被释放。用`'a`和`'b`分别表示`x`和`y`的生命周期，可以看到`x`活得更久，生命周期更长：

```rust
fn main() {
    let x;                // -------------------+--'a：x的生命周期
    {                     //                    |
        let y = 42;       // -+-- 'b：y的生命周期 |
        x = &y;           //  |                 |
    }                     // -+                 |
    println!("x: {}", x); //                    |
}
```

#### 函数中的生命周期

大部分时候生命周期是隐含并可以推断的，但是有时候编译器推断不了，比如下面的这个函数：

```rust
// 无法编译通过
fn longest_word(x: &String, y: &String) -> &String {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

编译器不知道返回值的生命周期，我们也不能确定，因为根据输入的值，x、y均有可能。对于这种函数，我们需要添加**生命周期注解**，来显式地声明返回值和函数参数的生命周期之间的关系：

```rust
fn longest_word<'a>(x: &'a String, y: &'a String) -> &'a String {
    ...
}
```

其中，`'a`就是生命周期注解，其名称无所谓，`'b`，`'xyz`都行，只需要**保证函数参数和返回值的生命周期注解一样**即可。函数后面`<'a>`的意思是该函数存在生命周期注解，名称为`'a`。后面带`'a`的参数说明这些参数的生命周期和返回值的生命周期相关。

上面代码的意思就是告诉编译器，`longest_word`这个函数的返回值的生命周期，和`x`、`y`这两个参数的生命周期有关。这样，Rust编译器在编译的时候，就会自动取有注解的参数的生命周期中**较短的那个作为返回值的生命周期**。这样，就可以检查出下面的错误：

```rust
// 编译不通过
fn main() {
    let magic1 = String::from("a!");
    let result;
    {
        let magic2 = String::from("shazam!");
        result = longest_word(&magic1, &magic2); // result可能是 magic2
    } // magic2被释放
    println!("The longest magic word is {}", result); // 避免了此处可能存在的悬垂指针
}
```

#### 结构或枚举中的生命周期

如果一个struct或一个枚举包含引用类型的字段，那么它也需要像函数一样**显式地标记生命周期**：

```rust
struct Highlight(&str); // 错误
struct Highlight<'a>(&'a str); // 正确
```

这里，`'a`可以认为是一个提醒，提醒 `Highlight` 结构的生存期不能超过它借用的 `&str` 的生存期。下面的代码可以解释一下：

```rust
#[derive(Debug)]
struct Highlight<'document>(&'document str);

fn erase(_: String) { }

fn main() {
    let text = String::from("The quick brown fox jumps over the lazy dog.");
    let fox = Highlight(&text[4..19]);
    let dog = Highlight(&text[35..43]);

    let moved_text = text; // 编译不通过，不能把 text 移走，因为 Highlight类型的对象（fox,dog）还在其生命周期内

    println!("{:?}", fox);
    println!("{:?}", dog);
}
```



