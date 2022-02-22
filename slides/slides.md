---
title: Async Programming in Rust
revealOptions:
    transition: 'slide'
---

## Agenda

* Rust async/.await
* Tokio async runtime
* Tokio debugging tools

---

# What is Async Programming?

----

## Simple Definition

* A system for efficiently calling functions which may return their result some time in the future

----

## Why?

* Blocking threads is evil
* OS threads are expensive
* There could be *thousands* of blocking IO calls

----

## Solution

* "Green Threads"
  * Running multiple tasks on the same OS thread
  * Requires an async "runtime" managing tasks
  * Similar to the task/thread scheduler of the OS
    * ...but simpler
    * Cooperative instead of preemptive multitasking

---

# Async Building Blocks

----

* Base building blocks are provided by Rust
  * ...and the Rust standard library
* Rust does *not* include an async runtime
  * Tokio is one crate implementing an async runtime

----

## The "Future" trait

* Central component of async programming in rust
* For polling functions which may return a value now or in the *future*

----

### Simplified Future Trait

    trait SimpleFuture {
        type Output;
        fn poll(&mut self, wake: fn()) -> Poll<Self::Output>;
    }

    enum Poll<T> {
        Ready(T),
        Pending,
    }

----

## async/.await

* "Look and Feel" of ordinary synchronous programming

----

### From:

    let future = client
        .get("http://my_server.com")
        .then(|res| match res {
            Err(_) => /* handle error */,
            OK(val) => /* do something */,
        });
    block_on(future);

----

### To:

    let res = client
        .get("http://my_server.com")
        .await;

    match res {
            Err(_) => /* handle error */,
            Ok(val) => /* do something */,
        };

----

## async

Creates Futures out of:
* functions
* code blocks
* closures

----

### functions

    async fn query_client {
        let res = client
            .get("http://my_server.com")
            .await;

        match res {
                Err(_) => /* handle error */,
                Ok(val) => /* do something */,
        };
    }

----

### async blocks

    async {
        // ...
        println!("Hello Tokio!");
    }.await

----

### closures

    let sum = async move |x, y| x + y;
    let result = sum(3, 4).await;

* Still unstable feature
* Requires the use of "move"

----

## .await

* Possibly suspending execution of the async construct
* Possibly moving between threads
* Not safe to use Types not supporting the "Send" trait
* Avoid holding Locks (e.g. Mutex) across .await calls

----

## Async lifetimes

* Async functions return a “Future”
  * Returned Future bound by the lifetime of the function arguments
* Use “move” keyword
  * Async blocks and closures allow the “move” keyword to take ownership of captured variables
* Use types supporting the “Send” and “Sync” traits

----

## Send and Sync Traits

* A type is “Send” if it is safe to send to another thread
  * Passing ownership between threads, *non-overlapping* use
* A type is “Sync” if it is safe to share between threads
  * Overlapping use on multiple threads
* A type is “Sync” if and only if &type is “Send”
  * Interesting example: MutexGuard is not “Send”(cannot be freed on a different thread), but it is “Sync”, because a reference to MutexGuard is “Send”

---

# Tokio

* First version released in 2016
  * 1.0 release in 2020
* De-facto standard asynchronous runtime for Rust
* Contributions from Google, Mozilla, Dropbox, AWS
  * Alice Ryhl (Google) main developer

----

## Key Features

* Fast
  * Zero-cost abstraction over the OS, bare metal performance
* Reliable
  * Widely used and mature
* Scalable
  * Minimal footprint, handles backpressure and cancellation

----

## Components

* A multithreaded, work-stealing task scheduler
  * Similar to rayon, but optimized for short tasks
  * Cooperative
  * Keep below 10-100 microseconds per task
* Reactor backend backed by native OS event queue
* Asynchronous TCP and UDP sockets

----

## Async main

* Use #[tokio::main] attribute
  * Can be used on any function
* Starts the function with a tokio runtime if called
  * Allows configuration of the runtime
  * Examples: <code>flavor = "current_thread"</code>
    * Starts a single-threaded runtime on the current thread

----

### Async main example

    #[tokio::main(flavor = "current_thread")]
    async fn main() -> Result<(), Box<dyn std::error::Error>> {
        let listener = TcpListener::bind("127.0.0.1:8080").await?;
        loop {
            let (mut socket, _) = listener.accept().await?;
            ...
        }
    }

----

### Other examples:

* <code>worker_threads = 2</code>
  * For limiting OS threads for multithreaded runtime
* <code>thread_name_fn</code>
  * Supply a function giving names to OS threads
* <code>thread_stack_size</code>
  * Specify the size of the stack for spawned threads

----

## Async unit tests

* Use #[tokio::test] attribute

----
### Example

    #[tokio::test]
    async fn server_with_mpsc() {
        let (tx, mut rx) = mpsc::channel(100);
        run(tx).await;

        assert_eq!(Some("hello".to_string()), rx.recv().await);
        assert_eq!(None, rx.recv().await);
    }

---

# Ecosystem

----

## Crates

* mio - "Metal IO"
  * Fast, low-level IO library, used by Tokio
* tokio-metrics
  * Runtime and per-task metrics
* tracing
  * Logging for async applications
* bytes
  * Utilities for working with byte buffers

----

# Tools

* Tokio Console 
  * A "task manager" for applications using tokio
  * Terminal/text based, but visual
  * For diagnostics and debugging
* PROST!
  * A protocol buffer implementation for Rust

---

# Questions?