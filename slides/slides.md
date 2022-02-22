---
title: Async Programming in Rust
revealOptions:
    transition: 'slide'
---

# What is Async Programming?

----

## Simple Definition

* Calling functions which may return their result some time in the future

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

3 ways:
* function
* capture
* block

----

What is it?
* Generates a Future keeping the function state

----
## functions

    pub enum Result<T, E> {
        /// Contains the success value
        Ok(T),
        /// Contains the error value
        Err(E),
    }

----

## await

* Manages state across await calls

----

Cons:
* Set of possible exceptions to handle unclear
  * Caller required to handle errors thrown x levels down the call stack
  * Can change due to changes in other parts of the code
* Can easily be ignored (in some languages)

---

# Tokio Runtime

* High Performance
* etc.

---

# Questions?