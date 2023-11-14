---
title: "Kokkos并行后端实现: C++ Thread"
date: 2023-11-13 22:20:57
tags: [Kokkos,HPC]
categories: [Tech,HPC,Kokkos]
---

# C++ Thread

C++ Thread是存在于C++ std库内的一个多线程库，该库是C++原生的一套做到了平台无关的多线程库。该库功能相对于市面上多线程计算库更为基础，提供了对线程的直接控制。

在Thread中创建一个线程通常如下所示

```c++
// thread example
#include <iostream>       // std::cout
#include <thread>         // std::thread
 
void foo() 
{
  // do stuff...
}

void bar(int x)
{
  // do stuff...
}

int main() 
{
  std::thread first (foo);     // spawn new thread that calls foo()
  std::thread second (bar,0);  // spawn new thread that calls bar(0)

  std::cout << "main, foo and bar now execute concurrently...\n";

  // synchronize threads:
  first.join();                // pauses until first finishes
  second.join();               // pauses until second finishes

  std::cout << "foo and bar completed.\n";

  return 0;
}
```

# Kokkos中的并行模型

Kokkos emploies task-centric parallel programming model. In this framework, each task has been split into numbers of `functor`, each functor acts as one of the smallest execution unit.

To launch a parallel task in Kokkos, here is example code:

```c++
#include <Kokkos_Core.hpp>

const size_t N = /* some data */;

typedef Kokkos::HostSpace ExecuteSpace;

int main(int argc, char **argv) {
  // ... init your own envs ...
  Kokkos.initialize(argc, argv);
  
  Kokkos::View<double *, ExecuteSpace> x("x", N);
  Kokkos::View<double *, ExecuteSpace> y("y", N);
  Kokkos::View<double *, ExecuteSpace> A("A", N);
  
  // ... init data here ...
  
  parallel_for("A=x+y", Kokkos::RangePolicy<ExecuteSpace>(0, N),
              KOKKOS_LAMBDA (const int i) {
                A(i) = x(i) + y(i);
              });
  
  Kokkos::finalize();
}
```

In this procedure, we declare the `Space` of each variable and `RangePolicy`, which tells our framework the expected execute place of kernel.

## How does this code run

We use `Thread` backend as example.

### Initialize

First of all, Kokkos will initialize thread pool before kernel running by invoking `initialize_space_factory`. Initialize code locates at `Kokkos_ExecSpaceManager.hpp`, this will invoke `ExecutionSpace::impl_initialize()`, while `ExecutionSpace` is the template parameter.

`initialize_space_factory` will be invoked in `Kokkos_<Runtime>Exec` automatically, after serval function calls, the actual useful function is `::Impl::<Runtime>Exec::initialize()`.

In `initialize()`, program will first check the basic information of current machine if `hwloc` is available. If `hwloc` is unavilable, thread count will be set to 1. Then program will initialize all threads by dispatching a noop task for each thread and spawn it. After thread received noop, thread status will be set to `ThreadsExec::Inactive` and wait next task.

### Task Dispatch

We use `parallel_for` as an example.

When we invoke `parallel_for`, it will use a class named `ParallelFor` to handle the execution. The entrance of parllel execution is `ParallelFor::execute()`, it will pass member function `exec` into execution environment `ThreadExec::start`.

Inside `ThreadExec::start()`, it will set current function to `exec` and notify threads by switching their state into `ThreadsExec::Active`, then threads will execute functions automatically.

To control the range each thread will execute, Kokkos provides two schedule type: static one and dynamic one, both of them share a same function name `exec_schedule`.

For static one, it will compute a work range and do a serial loop on it. After execute the functor, each thread will do `fan_in` operate and turn into `Inactive` status.

For dynamic one, it will leverage a work stealing algorithm to schedule work load. Each iteration will take `chunk_size()` tasks to execute, when this iteration has done, it will get next chunk's index and continue to execute.





