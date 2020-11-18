---
title: 信号量机制与管程
date: 2020-11-18 19:38:10
tags: 操作系统
---

## 信号量机制
信号量机制最早由提出图论的Dijkstra提出，其被广泛地使用于操作系统中用于解决同步问题。

### 信号量简介
信号量一般由以下几部分组成

- 一个整形```sem```  
通常```sem```的值等于共享资源的数量
- 两个操作
    - P操作(又名Wait)  
    P操作通常用于申请资源，其的主要过程一般为
        - ```sem -= 1```
        - 当```sem < 0```block当前进(线)程，并放入等待队列中
    - V操作(又名Signal)  
    V操作通常用于释放资源，其的主要过程一般为
        - ```sem += 1```
        - 当```sem <=0 ```时从等待队列中取出一个进(线)程，并唤醒

借助信号量机制，我们可以解决很多的多进(线)程间同步的问题。接下来我们将会使用几个例子来进一步地了解信号量的作用。

### 生产者-消费者问题
生产者-消费者问题是一个非常著名的多线程间同步的问题，接下来我们将会围绕这个问题探讨信号量机制的意义。

首先我们假设现在我们有一个盒子，x个消费者和y个生产者。并且我们规定同时可以有消费者从盒子中取出东西，同时只能由一位生产者向盒子中放入东西，且消费者和生产者同样不能同时从盒子中取出或者放入东西。

#### 情景0：多消费者-无生产者-盒子内的资源无限

情景0并不是定义上的生产者-消费者问题，这种情况只是提出用于具体了解信号量的工作方式的情景，故在本情景中我们假设同时只能有一位消费者从盒子中取出东西。  
这种情况实际上可以退化为多个等价的个体竞争一个资源的情况，由于我们规定了盒子同时只能由一位消费者进行访问，所以我们这里就可以使用一个信号量表示盒子的占用情况，由上文中的信号量的定义我们可以得知当任何消费者对盒子进行访问后下一次对```res.wait()```的调用就会将新的进程阻塞，直到访问盒子的消费者调用```res.signal()```进行唤醒。
```rust
let res: Semaphore = 1;

fn consumer(){
    res.wait();
    //对盒子中的资源进行消费
    res.signal();
}
```

#### 情景1：单消费者-单生产者-盒子容量为一

从情景1开始才是真正意义上的消费者生产者问题，但是这里我们还是限制了消费者和生产者的数量均为1。对于本情景中的条件，我们可以构造得到下述的伪代码
```rust
let full: Semaphore = 0;
let empty: Semaphore = 1;

fn consumer(){
    full.wait();
    //对盒子内的资源进行消费
    empty.signal();
}

fn provider(){
    empty.wait();
    //向盒子中生产内容
    full.signal();
}
```
从上面的代码就可以看到我们创建了两个信号量而非一个，原因也很简单，当盒子为空的时候消费者无法进行消费，当盒子为满的时候生产者无法进行生产，所以我们就可以让消费者对```full```作```wait()```标志空间已经被释放，同样生产者也可以使用```full.signal()```实现标志盒子中已经放入了资源。

#### 情景2：多消费者-单生产者-盒子容量为n

现在我们引入了多个消费者，我们知道当任何消费者在访问盒子的时候生产者都不能朝盒子中放入东西，但是我们只需要对上面的代码稍作修改即可解决这个问题。

```rust
let full: Semaphore = 0;
let empty: Semaphore = n;
let mutex: Semaphore = 1;
let using: Semaphore = 1;
let count: i32 = 0;

fn consumer(){
    full.wait();//当盒中有资源的时候
    mutex.wait();//当生产者未占用(盒子空闲)
    using.wait();
    count += 1;
    using.signal();
    //从此处才正式开始消费
    empty.signal();
    //从盒子中取出资源
    using.signal();
    using.wait();
    count -= 1;
    if count == 0{
        mutex.signal();//当所有消费者退出之后才释放mutex
    }
    using.signal();
    
}

fn provider(){
    empty.wait();//当盒子还有空间的时候
    mutex.wait();//当消费者未占用(盒子空闲)
    if count == 0{
        //向盒子中放入东西
        full.signal();
    }
    mutex.signal();   
}
```
从上面的代码我们可以看到这里我们加入了两个信号量，一个负责控制消费者和生产者分别对共享的盒子的访问，另外一个用于控制消费者的计数器。  
在上面的代码中我们实际上将多个消费者和单一生产者的问题抽象为了第一个消费者和生产者之间的问题，当消费者全部完成消费并退出之后我们才会释放掉```mutex```使得生产者可以向盒子中放入东西。同时务必要记住在多进(线)程对共享变量的操作中必须要使用信号量管理访问的权限。

## 管程
管程实际上是一种对同步锁抽象得到的数据结构，一般来说管程有以下几种特征
- 同时只允许一个进(线)程在管程中执行
- 使用条件变量管理管程内进(线)程的执行
- 拥有一个**入口队列**用于进(线)程排队进行管程  

实际上在本菜鸡看来管程的确有点难以理解，尤其是其的基本结构和几种不同的实现方式。

### 条件变量
管程采用条件变量控制已经进入管程的进程的执行，条件变量本身的各种特征又和信号量有点相似但是完全不同
- ```wait()```操作
    - 表示当资源被占用导致自身阻塞，并将自身放入等待队列
    - 执行```wait()```后将会从入口队列中唤醒一个新的进(线)程
- ```signal()```操作
    - 表示当前资源已经被使用完毕，唤醒等待队列中的一个进(线)程
    - 当等待队列为空的时候```signal()```操作没有意义

这里我们就可以清楚地看到条件变量和信号量的区别，对于条件变量来说由于调用其的进(线)程已经在管程中，故其的```wait()```和```signal()```无需成对，且在条件变量执行完成```wait()```操作后进(线)程必定阻塞，但是对于信号量的```V()```操作则不一定会阻塞。且在条件变量执行```wait()```操作或```signal()```的时候已经默认获得了互斥锁。

光谈概念感觉管程本身还是比较晦涩的，下面还是会用著名的生产者-消费者问题讨论管程

### 使用管程实现生产者-消费者问题

TODO:还没搞懂