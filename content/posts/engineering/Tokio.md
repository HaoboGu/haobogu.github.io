---
title: "Get Started With Tokio"
author: "Haobo Gu"
tags: [rust]
date: 2022-06-18T17:48:45+08:00
summary: Learn Tokio by Tokio official tutorial
---
# Get Started With Tokio 

## What's Tokio?

[Tokio](https://tokio.rs/) is an asynchronous runtime for the Rust programming language. It is designed for IO-bound applications, and provides many useful utilities for asynchronous cases.

## Tokio application
Let's start with a simple hello world tokio application. First of all, add tokio as the dependency of your project:

```toml
tokio = { version = "1", features = ["full"] }
```

Then, write your first tokio hello world program:

```rust
async fn say_world() {
    print!("world");
}

#[tokio::main]
async fn main() {
    // Calling `say_world()` does not execute the body of `say_world()`.
    let op = say_world();

    // This println! comes first
    print!("hello");

    // Calling `.await` on `op` starts executing `say_world`.
    op.await;
}
```

Here are some notes about this hello world program:

1. The `say_world` function is prefixed with `async` keyword, which indicates that it's an asynchronous function. In Rust, this asynchronous function will not be executed until `.await` is called.

2. The main function is also asynchronous, but it must be marked using `#[tokio::main]`.

3. At the beginning of `main`, `say_world` is called and the result is binded to `op`. However, because `say_world` is an asynchronous function, it won't be executed until `op.await` is called. Hence, the code above will always print `helloworld` at the console.

## Spawning
In this section, we'll write a server which accepts and processes TCP sockets.

In the main function, we bind a `tokio::net::TcpListener` to a local address, listen to the address and process incoming sockets in an infinite loop:

```rust
use tokio::net::{TcpListener, TcpStream};
#[tokio::main]
async fn main() {
    // Bind the listener to the address
    let listener = TcpListener::bind("127.0.0.1:6379").await.unwrap();

    loop {
        // The second item contains the IP and port of the new connection.
        let (socket, _) = listener.accept().await.unwrap();
        process(socket).await;
    }
}
```

Note that the `process()` function should be asynchronous:

```rust
async fn process(socket: TcpStream) {
    // process
}
```

The code above has a problem: we can only process only one request at a time. When it processes the request, it blocks in the inside the loop. In other languages, to process concurrent requests, threads/coroutines should be spawned to process requests in background, the main loop won't block. In Rust, you can do it as well, using `tokio::spawn`:

```rust
use tokio::net::TcpListener;
#[tokio::main]
async fn main() {
    let listener = TcpListener::bind("127.0.0.1:6379").await.unwrap();

    loop {
        let (socket, _) = listener.accept().await.unwrap();
        // A new task is spawned for each inbound socket. The socket is
        // moved to the new task and processed there.
        tokio::spawn(async move {
            process(socket).await;
        });
    }
}
```

You can see we spawned a closure(which is just like a anonymous function) using `tokio::spawn`. Note the `async` and `move` keyword before the spawned closure. `async` indicates that this closure is asynchronous and won't block, while `move` indicates that the used variables `socket` is moved into the closure and won't be dropped until the closure completes.

### Task
A Tokio task is an asynchronous green thread. It can be created using `tokio::spawn(async{})`. 

`tokio::spawn()` will return a `JoinHandle`, the caller can use `JoinHandle.await.unwrap()` to get the result of the task:

```rust
#[tokio::main]
async fn main() {
    let handle = tokio::spawn(async {
        // Do some async work
        "return value"
    });

    // Do some other work

    let out = handle.await.unwrap();
    println!("GOT {}", out);
}
```

### Data passed to the task
In the `TcpListener` example, we also add `move` keyword before spawning the task. This keyword is required, because `socket` cannot be accessed from multiple threads. 

### Task is `Send`!
Tasks spawned by tokio::spawn **must** implement `Send` trait. This is because when we call `.await`, the task is moved by tokio runtime between threads. 

But how to make a task `Send`? The answer is, if **all** data held by the task is `Send`, then the task is `Send`. If there exists which is not `Send`, the task won't compile, the following is an example:

```rust
use std::sync::{Mutex, MutexGuard};

// It won't compile!
async fn increment_and_do_stuff(mutex: &Mutex<i32>) {
    let mut lock: MutexGuard<i32> = mutex.lock().unwrap();
    *lock += 1;

    do_something_async().await;
} // lock goes out of scope here
```

This function will not compile, because `MutexGuard` is not `Send`. You can do a little refactoring to address it:

```rust
// This works!
async fn increment_and_do_stuff(mutex: &Mutex<i32>) {
    {
        let mut lock: MutexGuard<i32> = mutex.lock().unwrap();
        *lock += 1;
    } // lock goes out of scope here

    do_something_async().await;
}
```

## Share state between tasks
The state can be wrapped in `Arc<Mutex<_>>`, making it accessable across many tasks and potentially many threads. For example, if you want to pass a shared `HashMap` to spawned tasks, you can just use `Arc::new(Mutex::new(HashMap::new()))` to initialize the map:

```rust
use tokio::net::TcpListener;
use std::collections::HashMap;
use std::sync::{Arc, Mutex};

#[tokio::main]
async fn main() {
    let listener = TcpListener::bind("127.0.0.1:6379").await.unwrap();

    println!("Listening");

    let db = Arc::new(Mutex::new(HashMap::new()));

    loop {
        let (socket, _) = listener.accept().await.unwrap();
        // Clone the handle to the hash map.
        let db = db.clone();

        println!("Accepted");
        tokio::spawn(async move {
            process(socket, db).await;
        });
    }
}
```

Note that it is `std::sync::Mutex`, not `tokio::sync::Mutex` used to guard the `HashMap`.

## Channels
Channels pass informations between threads, which is similar to channels in Golang. But in Rust, there are more types of channels, which serve for different purposes:

1. [mpsc](https://docs.rs/tokio/latesttokio/sync/mpsc/index.html): multi-producer, single-consumer channel. Many values can be sent.

2. [oneshot](https://docs.rs/tokio/latesttokio/sync/oneshot/index.html): single-producer, single consumer channel. A single value can be sent.

3. [broadcast](https://docs.rs/tokio/latesttokio/sync/broadcast/index.html): multi-producer, multi-consumer. Many values can be sent. Each receiver sees every value.

4. [watch](https://docs.rs/tokio/latest/tokio/sync/watch/index.html): single-producer, multi-consumer. Many values can be sent, but no history is kept. Receivers only see the most recent value.

There are also some other channel crates in Rust ecosystem, such as `crossbeam`. [This document](https://docs.rs/tokio/latest/tokio/sync/mpsc/index.html#communicating-between-sync-and-async-code) explains the differences. In one word, `crossbeam::channel` would block the current thread while tokio not, because tokio is designed for asynchronous situations.

In this article, I'll introduce `mpsc` and `oneshot` as the example.

### Define a channel
First, we define a mpsc channel:

```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    // Create a new channel with a capacity of at most 32.
    let (tx, mut rx) = mpsc::channel(32);
}
```

`mpsc::channel(32)` returns both sender and receiver of the channel, we bind the sender with name `tx` and the receiver with `rx`. 32 is the buffer size of the channel.

To send a message to the channel, we can use `tx.send()`:

```rust
tokio::spawn(async move {
    tx.send("sending from first handle").await;
});
```

Because `mpsc` is a multi-producer channel, we can clone the sender and send messages from multiple tasks:

```rust
let tx2 = tx.clone();

tokio::spawn(async move {
    tx.send("sending from first handle").await;
});

tokio::spawn(async move {
    tx2.send("sending from second handle").await;
});
```

`.await` is called after `send()`, which indicates the sending thread will wait until the receiver reads the sent message.

Then, we use `rx.recv()` to receive messages sent to the channel:

```rust
while let Some(message) = rx.recv().await {
    println!("GOT = {}", message);
}
```

The receiver cannot be cloned, only one receiver can be used to receive messages for `mpsc` channel. When all senders are dropped, `rx.recv()` would return `None`, which indicates all senders are gone and the channel is closed.

The complete example code:

```rust
use tokio::sync::mpsc;

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel(32);
    let tx2 = tx.clone();

    tokio::spawn(async move {
        tx.send("sending from first handle").await;
    });

    tokio::spawn(async move {
        tx2.send("sending from second handle").await;
    });

    while let Some(message) = rx.recv().await {
        println!("GOT = {}", message);
    }
}
```

In the code, we process the received message in the main thread. You can also spawn a new thread to process it.

## IO
Tokio provides apis for asynchronous IO, which are similar with IO apis in `std`.

### Buffered read and write
Generally, the message is transmitted like a frame, a typical HTTP frame is like:

```rust
enum HttpFrame {
    RequestHead {
        method: Method,
        uri: Uri,
        version: Version,
        headers: HeaderMap,
    },
    ResponseHead {
        status: StatusCode,
        version: Version,
        headers: HeaderMap,
    },
    BodyChunk {
        chunk: Bytes,
    },
}
```

The frame will be converted to byte arrays to be transmitted, so when we read data, we have to convert byte arrays back to frames. Take `TcpStream::read()` as an example, when we manually call `read()`, an arbitrary amount of data might be returned. The returned data could be a frame, or a partial frame, or multiple frames. This is a little bit complex, as a result buffered IO is introduced to address this.

First of all, we wrap `TcpStream` with a buffer:

```rust
use bytes::BytesMut;
use tokio::net::TcpStream;

pub struct Connection {
    stream: TcpStream,
    buffer: BytesMut,
}
```

