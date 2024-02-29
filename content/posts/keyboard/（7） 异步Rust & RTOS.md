---
title: "Rust嵌入式教程（7）- RTOS概况"
author: "Haobo Gu"
tags: [rust]
date: 2024-02-26T11:13:40+08:00
summary: B站视频教程链接：https://www.bilibili.com/video/BV1G5411C7mu
---

> B站视频教程链接：https://www.bilibili.com/video/BV1G5411C7mu

## Rust + RTOS 概况

在Rust中，RTOS分为两类：

- 基于C bindings的RTOS
   - zephyr: [https://github.com/tylerwhall/zephyr-rust](https://github.com/tylerwhall/zephyr-rust)
   - FreeRTOS: [https://github.com/lobaro/FreeRTOS-rust](https://github.com/lobaro/FreeRTOS-rust)
- Pure Rust
   - 偏传统的RTOS
      - [https://github.com/tock/tock](https://github.com/tock/tock)
      - [https://github.com/oxidecomputer/hubris](https://github.com/oxidecomputer/hubris)
   - 异步运行时 👍
      - [https://github.com/embassy-rs/embassy](https://github.com/embassy-rs/embassy)
      - [https://github.com/rtic-rs/rtic](https://github.com/rtic-rs/rtic)
## Rust异步编程
### 简介
[async 编程入门 - Rust语言圣经(Rust Course)](https://course.rs/advance/async/getting-started.html)
### 异步运行时 embassy-rs
[Embassy](https://embassy.dev/)
## 示例代码（点灯 + button）

main.rs: 

```rust
#![no_std]
#![no_main]

use defmt::*;
use embassy_executor::Spawner;
use embassy_stm32::{
    exti::ExtiInput,
    gpio::{AnyPin, Input, Level, Output, Pin, Pull},
};
use embassy_time::Timer;
use {defmt_rtt as _, panic_probe as _};

// Declare async tasks
#[embassy_executor::task]
async fn blink(pin: AnyPin) {
    let mut led = Output::new(pin, Level::Low, embassy_stm32::gpio::Speed::High);

    loop {
        // Timekeeping is globally available, no need to mess with hardware timers.
        led.set_high();
        Timer::after_millis(150).await;
        led.set_low();
        Timer::after_millis(150).await;
    }
}

// Main is itself an async task as well.
#[embassy_executor::main]
async fn main(spawner: Spawner) {
    // Initialize the embassy-stm32 HAL.
    let p = embassy_stm32::init(Default::default());

    // Spawned tasks run in the background, concurrently.
    spawner.spawn(blink(p.PE3.degrade())).unwrap();

    let mut button = ExtiInput::new(Input::new(p.PC13, Pull::Down), p.EXTI13);
    loop {
        // Asynchronously wait for GPIO events, allowing other tasks
        // to run, or the core to sleep.
        button.wait_for_rising_edge().await;
        info!("Button pressed!");
        button.wait_for_falling_edge().await;
        info!("Button released!");
    }
}

```

build.rs:

```rust
fn main() {
    println!("cargo:rustc-link-arg-bins=--nmagic");
    println!("cargo:rustc-link-arg-bins=-Tlink.x");
    println!("cargo:rustc-link-arg-bins=-Tdefmt.x");
}
```

.cargo/config.toml

```toml
[target.thumbv7em-none-eabihf]
runner = "probe-rs run --chip STM32H7B0VBTx"

[build]
target = "thumbv7em-none-eabihf"

[env]
DEFMT_LOG = "info"
```

Cargo.toml:

```toml
[package]
edition = "2021"
name = "embassy-stm32h7-examples"
version = "0.1.0"
license = "MIT OR Apache-2.0"

[dependencies]
embassy-stm32 = { version = "0.1.0", features = ["defmt", "stm32h7b0vb", "time-driver-any", "exti", "memory-x", "unstable-pac", "chrono"] }
embassy-executor = { version = "0.5.0", features = ["task-arena-size-32768", "arch-cortex-m", "executor-thread", "defmt", "integrated-timers"] }
embassy-time = { version = "0.3.0", features = ["defmt", "defmt-timestamp-uptime", "tick-hz-32_768"] }
defmt = "0.3"
defmt-rtt = "0.4"
cortex-m = { version = "0.7.6", features = ["inline-asm", "critical-section-single-core"] }
cortex-m-rt = "0.7.0"
panic-probe = { version = "0.3", features = ["print-defmt"] }

# cargo build/run
[profile.dev]
codegen-units = 1
debug = 2
debug-assertions = true # <-
incremental = false
opt-level = 3           # <-
overflow-checks = true  # <-

# cargo build/run --release
[profile.release]
codegen-units = 1
debug = 2
debug-assertions = false # <-
incremental = false
lto = 'fat'
opt-level = 3            # <-
overflow-checks = false  # <-
```
