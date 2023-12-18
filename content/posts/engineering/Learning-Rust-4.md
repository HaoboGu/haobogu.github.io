---
title: "Learning Rust - 4"
author: "Haobo Gu"
tags: [rust]
date: 2021-07-09T10:18:19+08:00
summary: Module and testing in Rust
draft: false
---

# Learning Rust - 4

## Rust的代码组织

首先了解一些概念：

- Package：可以理解为一个代码库，包含一个或多个crate，以及如何生成这些crate的信息（cargo.toml）
- Crate：编译单元，每个crate可编译成一个可执行文件或一个库（lib）。Crate中包含隐式的顶层Module
- Module：代码组织的单位，代码模块

### Package和Crate

使用`cargo new my-project`生成的就是一个Package。在默认生成的工程中，有一个`cargo.toml`，这个就是Package的配置。此外，默认还有一个`src/main.rs`，这个就是当前Package的默认Crate。如果一个Package中包含`src/main.rs`和`src/lib.rs`，那么这个包里面就有两个Crate，一个是可执行文件的，一个是库的，他们的名称都和Package相同。

如果你还想添加更多的可执行文件的Crate，那么就在`src/bin`目录下添加文件即可，每个文件都是一个单独的Binary Crate。

### Module

Module是Rust中代码组织的**逻辑单元**。在一个模块中，可以声明很多东西：

- 常量
- 类型别名
- 函数
- 结构
- 枚举
- Traits
- `impl` 块
- 其他模块

在模块中，还能设置其成员的可见性，如果有`pub`修饰，那么就是模块外部可见的，否则不可见。下面是一个例子：

```rust
mod authentication {
    pub struct User {
        username: String,
        password_hash: u64,
    }

    impl User {
        pub fn new(username: &str, password: &str) -> User {
            User {
                username: username.to_string(),
                password_hash: hash_password(password),
            }
        }    
    }
    fn hash_password(input: &str) -> u64 { /*...*/ }
}

fn main() {

    let user = authentication::User::new("jeremy", "super-secret");

    println!("The username is: {}", user.username);
    println!("The password is: {}", user.password_hash);

}
```

模块`authentication`中有一个结构体`User`和一个函数`has_password`，然后`User`还定义了一个`new`函数。注意下面调用时，调用`new`函数使用的是`authentication::User::new()`，`::`符号就是用来获取module或struct的成员的。

#### `::`和`.`的区别

当左边是一个module或者一个struct类型的时候，使用`::`，当左边是一个值的时候，使用`.`。

### 添加三方Crate

如果想使用三方Crate，直接在`cargo.toml`里面添加dependency即可：

```toml
[dependencies]
regex = "1.4.2"
```

上面就是添加了一个名为regex的三方Crate，版本为1.4.2。

在代码中，就可以使用`use`关键字来使用Crate中的结构、Module等：

```rust
use regex::Regex;

fn main() {
    let re = Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap();
    println!("Did our date match? {}", re.is_match("2014-01-01"));
}
```

## Test

### 单元测试

在Rust中，可以使用`#[test]`标注单元测试：

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[test]
fn add_works() {
    assert_eq!(add(1, 2), 3);
    assert_eq!(add(10, 12), 22);
    assert_eq!(add(5, -2), 3);
}
```

然后，使用`cargo test`可以运行所有的单元测试。

如果我希望我的单元测试是失败的，或者测试某种产生panic的情况，可以使用`#[should_panic]`标注：

```rust
#[test]
#[should_panic]
fn add_fails() {
    assert_eq!(add(2, 2), 7);
}
```

如果希望跳过某些单元测试，则需要使用`#[ignore]`：

```rust
#[test]
#[ignore = "not yet reviewed by the Q.A. team"]
fn add_negatives() {
    assert_eq!(add(-2, -2), -4)
}
```

### 测试模块

一般情况下，需要把单元测试都写入测试模块，如果一个Module是测试模块，那么可以使用`#[cfg(test)]`标注：

```rust

fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod add_function_tests {
    use super::*;

    #[test]
    fn add_works() {
        assert_eq!(add(1, 2), 3);
        assert_eq!(add(10, 12), 22);
        assert_eq!(add(5, -2), 3);
    }

    #[test]
    #[should_panic]
    fn add_fails() {
        assert_eq!(add(2, 2), 7);
    }

    #[test]
    #[ignore]
    fn add_negatives() {
        assert_eq!(add(-2, -2), -4)
    }
}
```

测试模块会在`cargo test`时编译并且运行

### 文档测试

Rust中有一种注释叫做文档注释，以`///`标记。文档注释的所有内容会被写入Markdown中，并且文档注释中的Markdown代码块可以被编译和测试，这就是文档测试：

```rust
/// Generally, the first line is a brief summary describing the function.
///
/// The next lines present detailed documentation. 
/// Code blocks start with triple backticks. The code has an implicit `fn main()` inside and `extern crate <cratename>`,  
/// which means you can just start writing code.
///
/// ```
/// let result = basic_math::add(2, 3);
/// assert_eq!(result, 5);
/// ```
///
/// ```rust,should_panic
/// panic!("panic");
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
```

使用`cargo test`同样会跑文档注释中的代码块

### 集成测试

Rust还支持把Crate作为一个整体来测试，这些测试在单独的目录和文件中：`tests/xxx.rs`。
