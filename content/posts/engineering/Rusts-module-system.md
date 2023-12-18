---
title: "Rust's Module System"
author: "Haobo Gu"
tags: [Rust]
date: 2021-07-21T10:19:31+08:00
summary: Learn Rust's Module System
draft: false
---

Original post: http://www.sheshbabu.com/posts/rust-module-system/

Rust的模块系统比较特殊，初学者不太容易上手理解，上面的文章详细介绍了Rust模块系统，下面是阅读笔记。

## 工程结构

首先，我们看一个典型的Rust工程的文件结构，和其中的调用

![image-20210721103030139](https://www.sheshbabu.com/images/2020-rust-module-system/rust-module-system-1.png)

## Example 1

首先看第一个例子：从`main.rs`中调用`config.rs`。在Rust中，**每一个文件或文件夹**都被看做一个`module`，如`config.rs`，实际上就是一个名为`config`的module。我们可以从其他的文件中导入该module。在最初没有任何导入语句的时候，Rust编译器只能看到如下的结构：

<img src="https://www.sheshbabu.com/images/2020-rust-module-system/rust-module-system-2.png" alt="image-20210721103513662" style="zoom:50%;" />

默认情况下，Rust的编译器只会把`main.rs`看做是`crate`模块，而不会根据文件目录自动生成模块树，所有的模块结构都需要我们手动导入构建。

那么，接下来我们就手动构建我们的模块树。在Rust中，关键字`mod`被用来声明一个模块或者子模块。比如，对于`config.rs`，我们需要使用`mod config;`来声明该模块。

容易出错的是，我们不是在`config.rs`中使用`mod config;`声明，而是需要在`main.rs`中使用`mod config;`去声明config模块：

```rust
// main.rs
mod config; // declare config mod
```

在`main.rs`中声明了config模块之后，Rust的编译器就会去寻找`config.rs`文件或者`config/mod.rs`文件来构建模块结构。如果`config.rs`中有一个名为`print_config()`的函数，那么就可以通过`config::print_config()`来调用了。下面是完整的例子：

```rust
// main.rs
mod config;

fn main() {
  config::print_config();
  println!("main");
}

// config.rs
pub fn print_config() {
  // 注意需要把函数声明为public
  println!("config");
}
```

到此为止我们学会了同级的模块导入。但是在实际工程中，我们往往希望通过文件夹来把一个系列的文件组织到一起。下面看第二个例子。

## Example 2

假设我们需要从`main.rs`中调用`routes/health_route.rs`中的函数`print_health_route`。由于`health_route.rs`在文件夹`routes`下面，因此这个文件对`main.rs`来说是不可见的。如何解决呢？这个时候，我们就需要在`routes`文件夹下面添加一个名为`mod.rs`的文件。

```diff
my_project
├── Cargo.toml
└─┬ src
  ├── main.rs
  ├── config.rs
  ├─┬ routes
+ │ ├── mod.rs
  │ ├── health_route.rs
  │ └── user_route.rs
  └─┬ models
    └── user_model.rs
```

`mod.rs`在Rust的模块系统中非常重要。还记的上面的`config.rs`吗？如果config模块的代码越来越多，需要分成若干个文件进行组织的时候，就需要创建`config`文件夹，然后在`config/mod.rs`中声明文件夹下的所有子模块了。可以这样认为：文件夹模块的`config/mod.rs`其实就相当于单文件模块的`config.rs`。

在本例子中，也是一样的。想要在`main.rs`中调用`print_health_route()`，我们需要做如下三件事：

1. 创建`routes/mod.rs`，并且在`main.rs`中，使用`mod routes;`声明routes模块
2. 在`routes/mod.rs`中声明`health_route`子模块，并且把它声明为public
3. 在`routes/health_route.rs`中，把函数`print_health_route()`声明为public

完成的代码如下：

```rust
// main.rs
mod config;
mod routes;

fn main() {
  routes::health_route::print_health_route();
  config::print_config();
  println!("main");
}

// routes/mod.rs
pub mod health_route;

// routes/health_route.rs
pub fn print_health_route() {
  println!("health_route");
}
```

此时的模块树如下所示：

<img src="https://www.sheshbabu.com/images/2020-rust-module-system/rust-module-system-4.png" alt="image-20210721105947578" style="zoom:50%;" />

## Example 3

第三个例子中，我们会尝试如下的调用链路：`main.rs => routes/user_route.rs => models/user_model.rs`。

首先和Example 2中的一样，我们把`models/user_model.rs`子模块构建好：

<img src="https://www.sheshbabu.com/images/2020-rust-module-system/rust-module-system-5.png" alt="image-20210721110324269" style="zoom:50%;" />

然后，我们看调用链路的后半部分：从`routes/user_route.rs`中调用`models/user_model/rs`中的函数。对于同级的函数调用，我们可以从模块树的最上面逐级往下调用，即`crate::models::user_model`

```rust
// routes/user_route.rs
pub fn print_user_route() {
  crate::models::user_model::print_user_model();
  println!("user_route");
}
```

但是，在文件路径非常长的时候，每次从根模块往下找路径会非常冗长，因此Rust提供了`super`关键词来定位到父模块。

如果我想想要从`user_route.rs`中调用`health_route`，那么可以使用`super::health_route::print_health_route()`代替`crate::routes::health_route::print_health_route`：

```rust
pub fn print_user_route() {
  // crate::routes::health_route::print_health_route();
  // can also be called using
  super::health_route::print_health_route();

  println!("user_route");
}
```

## Use关键字

在调用其他模块时，大部分时候不会每次都写完整的调用链路，而是会使用`use`关键字先导入某个模块。就上面的路子，可以改成如下的代码：

```rust
// 导入
use crate::routes::health_route::print_health_route;

pub fn print_user_route() {
  // 使用
  print_health_route();
  println!("user_route");
}
```

## 三方包

在使用三方包时，需要在`Cargo.toml`中声明`[dependencies]`。声明之后的三方包模块会在工程所有的模块中自动生效。

举例：如果我们在`[dependencies]`中使用了`rand`包，在任意一个文件中，我们可以直接使用该模块：

```rust
pub fn print_health_route() {
  // 直接使用即可
  let random_number: u8 = rand::random();
  println!("{}", random_number);
  println!("health_route");
}
```

当然，我们也可以先`use`，然后再使用：

```rust
use rand::random;

pub fn print_health_route() {
  let random_number: u8 = random();
  println!("{}", random_number);
  println!("health_route");
}
```

## 总结

1. Rust中的模块系统需要显式地去声明结构
2. 在声明一个模块时，需要在它的父级声明，而不是在这个文件本身（上面的`main.rs`相当于是`config.rs`的父级，同样，`mod.rs`也是相当于是同级的`health_score.rs`的父级）
3. `mod`关键字用于声明子模块
4. 如果想要一个函数、结构体等被其他模块使用，必须使用`pub`关键字声明其为public
5. 使用`use`关键字可以导入模块以减少重复代码
6. 三方包不必显式地声明

