---
title: "Rust Build Scripts"
author: "Haobo Gu"
tags: [rust]
date: 2021-11-12T17:27:44+08:00
summary: Introduction to Rust's build scripts
---

# Rust Build Scripts
https://doc.rust-lang.org/cargo/reference/build-scripts.html

## What's Build Scripts in Rust
Build scripts in Rust is used to integrate Rust code to external tools or dependecies such as C libraries, or execute tasks before compiling the Rust project. You can enable build scripts by placing a file named `build.rs` at the root of a Rust package. Cargo will compile and execute the build script before building the package.

Here are some usages of build scripts according to the Cargo Book:
> 1. Building a bundled C library.
> 2. Finding a C library on the host system.
> 3. Generating a Rust module from a specification.
> 4. Performing any platform-specific configuration needed for the crate.

The following sections will introduce the basics of build scripts in Rust.

## Life Cycle of Build Scripts
Build scripts are compiled to an executable and executed before the package is built. 
It should return a zero exit code if there's no error. The rest of the package will be compiled after the build script finished.
If any error occurs, a non-zero exit code should be returned by the build script to interrupt the build process. 

By default, Cargo will rerun the build script if any file changes in the project. But you can also specify the watched files using `rerune-if-changed` command:
```rust
println!("cargo:rerun-if-changed=wrapper.h");
```	

## Inputs and Outputs of the Build Script
All inputs of the build script are passed using environment variables. [Here](https://doc.rust-lang.org/cargo/reference/environment-variables.html#environment-variables-cargo-sets-for-build-scripts) is the list of all available inputs for build scripts.

All output files of the build script should be saved in the directory specified in `OUT_DIR` environment variable. 
The script may commnunicate with Cargo by printing commands like `cargo:xxx` to stdout, all other lines will be ignored.

Here is a list of all commands that Cargo accepts according to the Cargo Book:

- cargo:rerun-if-changed=PATH — Tells Cargo when to re-run the script.
- cargo:rerun-if-env-changed=VAR — Tells Cargo when to re-run the script.
- cargo:rustc-link-arg=FLAG – Passes custom flags to a linker for benchmarks, binaries, cdylib crates, examples, and tests.
- cargo:rustc-link-arg-bin=BIN=FLAG – Passes custom flags to a linker for the binary BIN.
- cargo:rustc-link-arg-bins=FLAG – Passes custom flags to a linker for binaries.
- cargo:rustc-link-lib=[KIND=]NAME — Adds a library to link.
- cargo:rustc-link-search=[KIND=]PATH — Adds to the library search path.
- cargo:rustc-flags=FLAGS — Passes certain flags to the compiler.
- cargo:rustc-cfg=KEY[="VALUE"] — Enables compile-time cfg settings.
- cargo:rustc-env=VAR=VALUE — Sets an environment variable.
- cargo:rustc-cdylib-link-arg=FLAG — Passes custom flags to a linker for cdylib crates.
- cargo:warning=MESSAGE — Displays a warning on the terminal.
- cargo:KEY=VALUE — Metadata, used by links scripts.

## Build Dependencies
You can specify dependencies for build scripts using `build-dependencies` section in Cargo.toml:
```toml
[build-dependencies]
cc = "1.0.46"
```
The build script cannot use dependencies or dev-dependencies in Cargo.toml.

## The `package.links` Manifest Key
The `package.links` manifest key is used to specify the native libraries that the package depends on. When using this key, the package must have a build script with the `cargo:rustc-link-lib=[KIND=]=NAME` command. With `links` key, Cargo knows about the native libraries that the package depends on. Also, metadata is passed as the output of build script in the `KEY=VALUE` format using `links` key(see [cargo:KEY=VALUE](https://doc.rust-lang.org/cargo/reference/build-scripts.html#the-links-manifest-key) command). This metadata is automatically converted to an environment variable for the **dependent's** build script. 
```toml
[package]
# links to libonnxruntime
links = "onnxruntime"
```
By default, there is at most one package per `links` value to prevent conflict symbols. 

## `*-sys` Packages
In Rust's convention, any package that link to system libraries should have a `-sys` suffix. In our case, we generated `onnxruntime-sys` at 【TODO】. It's also common to have another package which wraps the `-sys` package, for example, `onnxruntime`, to provide safe and high-level APIs.

## Overriding the Build Script
You can also override params in build script at [Cargo configuration file](https://doc.rust-lang.org/cargo/reference/config.html). Here is an example:
```toml
[target.x86_64-unknown-linux-gnu.foo]
rustc-link-lib = ["foo"]
rustc-link-search = ["/path/to/foo"]
rustc-flags = "-L /some/path"
rustc-cfg = ['key="value"']
rustc-env = {key = "value"}
rustc-cdylib-link-arg = ["…"]
metadata_key1 = "value"
metadata_key2 = "value"
```
In this case, if a package links to `foo`, then it's build script will **not** be executed. Instead, the medata specified in Cargo configuration file will be used.

## Learn Build Script from Examples
The Cargo book provides a complete [example](https://doc.rust-lang.org/cargo/reference/build-script-examples.html) of how to write build scripts.
