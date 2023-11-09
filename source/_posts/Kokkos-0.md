---
title: "高性能的异构并行框架Kokkos: 0"
date: 2023-11-09 14:07:11
tags: [Kokkos,HPC]
category: [Tech,HPC,Kokkos]
---

# 什么是Kokkos

Kokkos是一个跨平台的高性能并行编程框架，其提供了一套能够在多个不同后端执行并行代码的运行时，使得单一源代码可以在不修改或只做少量修改的情况下在多种不同的平台上高性能地运行。

## Kokkos支持的平台

| 平台   | 并行框架                                    |
| ------ | ------------------------------------------- |
| CPU    | Serial,OpenMP,C++ Thread,SYCL(Experimental) |
| GPU    | CUDA,ROCm,SYCL                              |
| Others | HIP,HPX                                     |

# 安装Kokkos

通常情况下使用Kokkos有两种方式：第一种方式为将Kokkos安装到系统中，使用CMake的`find_package()`来搜索和链接Kokkos；第二种方式为将Kokkos作为项目的一部分，与项目共同编译，此处将会介绍第一种形式。

第一步自然是需要获取Kokkos的源代码，此处建议前往Kokkos的GitHub获取Release代码。

获取程序代码后进入Kokkos源代码目录，建议编译文件夹，此处假设为`build`

```bash
mkdir build
cd build
# Suppose we enable OpenMP&Serial for CPU, CUDA for GPU.
# For each platform, we can only enable ONE framework.
# Please choose proper compiler for your environment.
cmake .. -DKokkos_ENABLE_CUDA=ON -DKokkos_ENABLE_OpenMP=ON -DKokkos_ENABLE_Serial=ON \\
				 -DCMAKE_INSTALL_PREFIX=/path/to/install -DCMAKE_CXX_COMPILER=g++ \\
				 -DCUDA_ROOT=/path/to/cuda
# Build and install
make -j32 && make install
```

# 一个Kokkos示例程序

此时若需要在程序中引入Kokkos，可以使用CMake寻找系统中的Kokkos并与程序进行链接

```cmake
# ...
find_package(Kokkos REQUIRED)
# find other libraries
# add your source code
target_link_library(myTarget Kokkos::kokkos)
```

```c++
#include <Kokkos_Core.hpp>

const int N = 10;
int main(int argc, char **argv) {
  /* --- init program --- */
  /* ---     ....     --- */
  Kokkos::initialize(argc, argv);
  /* ---     ....     --- */
  
  /* --- declare datatype --- */
  typedef Kokkos::OpenMP ExecutionSpace;
  typedef Kokkos::View<double*, ExecutionSpace> MyView;
  
  MyView x	("x"	, N);
  MyView y	("y"	, N);
  MyView res("res", N);
  
  /* --- init data --- */
  for (int i = 0; i < N; i ++ ){
    x(i) = i; y(i) = i;
  }
  
  /* --- compute --- */
  Kokkos::parallel_for("x+y", RangePolicy<ExecutionSpace>(N),
                      KOKKOS_LAMBDA (const int i) {
                        res(i) = x(i) + y(i);
                      });
  
  /* --- print result --- */
  for (int i = 0; i < N; i ++) {
    std::cout  << res(i) << ' ';
  }
  /* --- finalize --- */
  Kokkos::finalize();
}
```

编译，并运行该程序能得到如下的结果

```bash
$ ./myTarget
> 2 4 6 8 10 12 14 16 18 20
```

