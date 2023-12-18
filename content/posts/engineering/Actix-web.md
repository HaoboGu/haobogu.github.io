---
title: "Basic Conceptions in actix-web"
author: "Haobo Gu"
tags: [rust]
date: 2021-11-16T17:29:10+08:00
summary: Basic Conceptions in actix-web
draft: true
---
# Basic Conceptions in actix-web
## What's actix-web
[actix-web](https://actix.rs/) is a high-performance web framework written in Rust. 

## A hello world example
Let's start from a hello world example:
```rust
use actix_web::{get, web, App, HttpResponse, HttpServer, Responder};

#[get("/")]
async fn hello() -> impl Responder {
    HttpResponse::Ok().body("Hello world!")
}

async fn manual_hello() -> impl Responder {
    HttpResponse::Ok().body("Hey there!")
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .service(hello)
            .route("/hey", web::get().to(manual_hello))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```
The code is quite simple. First, we define a `hello` service and a `manual_hello` handler. Then, in `main` function, a `HttpServer` is started and binded into `127.0.0.1:8080`.

We can test the code using `cargo run` and try `http://127.0.0.1:8080/` or `http://127.0.0.1:8080/hey` in your browser.

In the following sections, I'll introduce some basic concepts of `actix-web`.

## Actor
`actix` is based on the actor model, as well as `actix-web`. An actor is an concurrent computation unit, which receives message, do independent computation, and send message to other actors. An application can be written as a group of independent actors, which communicate with each other via messages.

Actor executes with a specific `Context<A>` in `actix-web`. The context is valid only when the actor is executing.

Any rust struct can be an actor, by implementing the `Actor` trait. To be able to process a message, the actor must implement the `Handler<M>` trait, where `M` is the message type. All messages are defined statically in your crate.

## Application
`actix-web` provides many useful components for building a modern web server, such as routing, middleware, request pre-processing, etc.

An application is a namespace where we can register routes for resources and middlewares. It also stores shared states of all request handlers.

In the example above, you can see that we register a route `\hey` and a service for our `App` instance.

## Server
The `HttpServer` is reponsible for processing HTTP requests, it can be binded to an address by `bind()`. You can also bind the server to ssl socket by `bind_openssl()`. You can use `run()` method to start the server, the `run()` method an instance of `Server` which provides several methods for managing the server, such as `pause()`, `resume()` and `stop()`.

In our example, we create a server and bind it to `127.0.0.1:8080`, so that we can visit the binded address in the browser.

## Handler
Handler is an async function which processes the incoming request. It accepts parameters which can be extracted from an HTTP request, and returns a type/struct which can be converted to an HTTP response. The return type of the handler should implement `Responder` trait. Then the `respond_to()` of `Responder` is called to convert the returned object to a `HttpResponse` or `Error`. `actix-web` provides built-in implementations for some standard types, such as `String`, `&'static str`, etc.

In our example, function `hello` and `manual_hello` are both handlers.

If you want to return your custom type from a handler, you can implement `Responder` trait for your type. Here is an example for an `application/json` response:
```rust
use actix_web::{Error, HttpRequest, HttpResponse, Responder};
use futures::future::{ready, Ready};
use serde::Serialize;

#[derive(Serialize)]
struct MyObj {
    name: &'static str,
}

// Implement Responder for MyObj
impl Responder for MyObj {
    type Error = Error;
    type Future = Ready<Result<HttpResponse, Error>>;

    // The `respond_to` method should be implmentated for every type which implements `Responder`
    fn respond_to(self, _req: &HttpRequest) -> Self::Future {
        let body = serde_json::to_string(&self).unwrap();

        // Create response and set content type
        ready(Ok(HttpResponse::Ok()
            .content_type("application/json")
            .body(body)))
    }
}

// This handler returns a custom type MyObj, which implements `Responder`
async fn index() -> impl Responder {
    MyObj { name: "user" }
}
```

## Extractor
The `Extractor` trait is used to extract parameters from the request. Also, `actix-web` provides some default extractor implementations.