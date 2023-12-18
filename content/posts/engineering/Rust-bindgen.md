---
title: "Rust bindgen "
author: "Haobo Gu"
tags: [rust]
date: 2021-11-11T16:06:12+08:00
summary: Introduction to Rust bindgen
---
# Rust bindgen

## What's bindgen?
[bindgen](https://github.com/rust-lang/rust-bindgen) is a tool which generates Rust FFI to C/C++ libraries automatically. It's quite useful when we want to use a C/C++ library in Rust. For example, PyTorch provides C library for users which don't want to use Python. With bindgen, we can quickly create Rust binding of PyTorch C library from C header(see [tch-rs](https://github.com/LaurentMazare/tch-rs))

## Use bindgen
The recommended way to use bindgen is using it in `build.rs`. `build.rs` is a Rust file placed in the root of a package which is used to integrate third-party libraries or user customized tools to Rust compiling process. Cargo will compile and execute `build.rs` first and then build the package.

Because many C/C++ headers have platform-specfic features, with using bindgen inside `build.rs`, we can generate bindings for the current target on-the-fly. Other users can also use your package by generating bindings for their platform.

In the following sections, we will take onnxruntime C library as an example, generate Rust bindings using bindgen.

### Add bindgen as a build dependency
First, we have to add bindgen as the build dependency. A build dependency is the dependency which is only used in building(see [this section](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html) in The Cargo Book).
```toml
[build-dependencies]
bindgen = "0.59.1"
```

### Create a wrapper.h Header
Because some libraries have more than one headers, we can include those headers in `wrapper.h` and take `wrapper.h` as an entrypoint for bindgen. 

In our example, onnxruntime's header file can be found at https://github.com/microsoft/onnxruntime/blob/master/include/onnxruntime/core/session/onnxruntime_c_api.h. So we can just download it and then use it in `build.rs`.

### Create build.rs file
Create `build.rs` at the project root, Cargo will automatically compile and execute it before compiling the rest of the project.

```rust
extern crate bindgen;

use std::env;
use std::path::PathBuf;

fn main() {
    // Tell cargo to invalidate the built crate whenever the wrapper changes
    println!("cargo:rerun-if-changed=onnxruntime_c_api.h");

    // The bindgen::Builder is the main entry point
    // to bindgen, and lets you build up options for
    // the resulting bindings.
    let bindings = bindgen::Builder::default()
        // The input header we would like to generate
        // bindings for.
        .header("onnxruntime_c_api.h")
        // Tell cargo to invalidate the built crate whenever any of the
        // included header files changed.
        .parse_callbacks(Box::new(bindgen::CargoCallbacks))
        // Finish the builder and generate the bindings.
        .generate()
        // Unwrap the Result and panic on failure.
        .expect("Unable to generate bindings");

    // Write the bindings to the src/bindings/[os]/[arch]/bindings.rs file.
    let out_path = PathBuf::from(env::var("CARGO_MANIFEST_DIR").unwrap())
        .join("src")
        .join("bindings")
        .join(env::var("CARGO_CFG_TARGET_OS").unwrap())
        .join(env::var("CARGO_CFG_TARGET_ARCH").unwrap());

    // If the directory doesn't exist, create it
    fs::create_dir_all(&out_path).expect("Unable to create dir");

    // Write bindings to file
    bindings
        .write_to_file(out_path.join("bindings.rs"))
        .expect("Couldn't write bindings!");
}
```
Run `cargo build`, and then the bindings to onnxruntime are generated, you can find it at `src/bindings/[os]/[arch]/bindings.rs`.

### Use generated bindings
`include!` macro can be used to dump the generated bindings into crate's main entry point.We can add different cfg headers for bindings of different platforms.

```rust
#![allow(non_upper_case_globals)]
#![allow(non_camel_case_types)]
#![allow(non_snake_case)]

#[cfg(all(target_os = "windows", target_arch = "x86_64"))]
include!(concat!(
    env!("CARGO_MANIFEST_DIR"),
    "/src/bindings/windows/x86_64/bindings.rs"
));

#[cfg(all(target_os = "macos", target_arch = "x86_64"))]
include!(concat!(
    env!("CARGO_MANIFEST_DIR"),
    "/src/bindings/macos/x86_64/bindings.rs"
));
``` 
Because onnxruntime's symbols are defined in C, they may not follow Rust's style convention. We can suppress warnings by a bunch of `#![allow(...)]` pragmas.

### Test bindings
The generated code contains some tests. We can also add our test at `src/lib.rs`:
```rust
#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn it_works() {
        assert_eq!(8, ORT_API_VERSION);
    }
}
```
Run `cargo test`, cargo will execute all tests defined in `bindings.rs` and `lib.rs`.
```
test bindgen_test_layout_wait__bindgen_ty_2 ... ok
test tests::it_works ... ok
test bindgen_test_layout_wait__bindgen_ty_1 ... ok

test result: ok. 81 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.01s
```

So far, we have successfully generated bindings for onnxruntime and tested them. With bindgen, we can also customize the generated bindings. For more options of bindgen, check https://rust-lang.github.io/rust-bindgen/customizing-generated-bindings.html.

## More about `build.rs`
We just generated bindings for onnxruntime. But that's not enough if we want to use it in production, especially when you want to deploy the application using the bindings to different platforms. This is where `build.rs` comes into play.

As we mentioned above, `build.rs` is compiled and executed before the package is compiled. So we can add somethings like platform-specific configuration, library downloading, etc. to `build.rs`. The full reference of `build.rs` is [here](https://doc.rust-lang.org/cargo/reference/build-scripts.html).


