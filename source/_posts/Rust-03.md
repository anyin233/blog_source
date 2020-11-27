---
title: 从零开始的Rust生活：03-控制流与逻辑运算符
date: 2020-11-27 22:23:00
tags: [Rust,程序语言]
---

# 前言

作为一款合格的程序语言，基本的控制语句是必须有的，而对于Rust来说她的控制流语句相对于传统的语言来说都要更为丰富，本文将会大体地介绍Rust的各类控制语句及其的基本用法。

# if

```if```语句作为大多数语言的标配，在Rust中自然也没有缺席，对于Rust来说与```if```搭配的同样是```else```，但是不同于经常拿来和Rust对比的C++，Rust的```if```语句后面的条件并不需要括号包裹起来，如果你用括号把它包裹的话编译器将会送给你一个大大的warning，所以说通常Rust中的一个条件语句就像这样
```rust
if condition {
    // do something
}else{
    // do something
}
```

# match

说到了```if```，自然少不了我们的好朋友```switch```，不过非常喜闻乐见的是Rust并没有```switch```，取而代之的是功能强大的```match```。```match```作为Rust中著名的条件选择语句，它不止用于多条件的判断控制，它还经常出现于```Option```的判断等等各类```None```类型或者错误处理的场景。常见的```match```的使用场景如下

```rust
match(item){
    condition_1 => //do something
    condition_2 => {
        //do something
    }
    _=> //do default thing
}
```
这里我们可以看到Rust的```match```语句支持单行或者多行的情况，其中多行的情况必须使用大括号包起来，同时```match```同样有一个```_```分支，这个分支一般是作为default分支存在。有一点非常重要，所有分支的返回值类型必须相同。

但是前面又提到了```match```相对于```switch```更为强大，所以这里我们将会探讨之所以```match```强大的原因

## match可以作为表达式

在Rust中，match有一个非常常用的用法，在这种情况下所有的分支都必须要有返回值，就像这样

```rust
let s = match (item){
    condition_1 => {
        1
    }
    condition_2 => {
        1 * 2
    }
    _=> {
        3
    }
}
```

在这种情况下各个代码块最终的执行后的返回值将会被赋给```s```

## match的分支条件可以用于解包各类enum

上面也说到了```match```可以用于处理```Option<T>```类型，在Rust中我们可以这样用

```rust
let s = Some(5);

match(s){
    Some(x) => {
        println!("{}", x);
    }
    None => {
        println!("is none");
    }
}// 输出5

```
就像这样，Rust可以利用```match```语句轻松地解决```Option<T>```类型的问题。

# 后记

在写了四篇Rust的博客之后，我突然发现自己对Rust的理解非常的零散肤浅，所以从本篇开始《从零开始的Rust生活》系列将会暂时停更，待到我更加了解Rust后再继续进行更新。接下来本博客将会开始逐步更新本人的Rust学习记录，同步于本人的Rust的学习进程。