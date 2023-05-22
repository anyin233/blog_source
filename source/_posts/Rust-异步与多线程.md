---
title: Rust 异步与多线程
date: 2022-02-18 21:38:28
tags: [Rust,程序语言,异步]

---

> 该文档为个人学习记录，可能存在大量错误

# 概述

Rust中std库实现异步与多线程的方式包括了`async/await`和`thread`，二者在官方文档中分别对应了`high-level`和`low-level`的场景。

<!--more-->

# Rust中的数据同步

实现多线程必须要考虑到数据同步的问题，Rust给出的解决方案是`Mutex<T>`和`Arc<T>`，二者可以分别对应到不具备线程安全的`RefCell<T>`和`Rc<T>`。两组容器的使用方式基本一致，通常情况下的一个使用实例是`Arc<Mutex<T>>`从而保证某个共享的数据可以存在多个所有者，同时实现内部可变性。

# std::thread

该实现方式为 *The Book* 中讲述的实现方式，最基本的代码实现形式为

```rust
let handler = thread::spawn(move || {
  // do something
});
```

上面的代码实现创建了一个新的线程，并返回对应线程的`handler`，基于对`handler`的控制可以实现对该线程的管理。其中`handler`具有`JoinHandler`类型，实现了三个有关多线程的方法，分别是`join()`，`thread()`和`is_running()`。其中`join()`方法使得执行该方法的线程会在执行到`handler.join()`时阻塞等待线程的退出，`thread()`返回`handler`对应的线程信息，`is_running()`则指示线程是否正在运行。

# async/await

**TODO**

