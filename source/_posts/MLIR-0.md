---
title: 使用MLIR编写一个自己的程序语言 Ep.-1
date: 2023-10-30 21:28:34
tags: [MLIR,Compiler]
categories: [Tech, Compiler, MLIR]
---

# 什么是MLIR

MLIR是LLVM项目的一部分，其目的在于减少在新的程序语言开发设计中所需要被重复实现的功能模块，从而实现低工作量且快速地开发新的程序语言或者为现有的程序语言进行优化。

# 基础概念

MLIR存在着一种叫作方言(Dialect)的概念。在MLIR中，所有的高级语言都会被转换为一个统一的中间表示，而MLIR为了便于开发者扩展这个中间表示，提出了一个方言的概念，通常一个mlir语句是这样的

```mlir
%t_tensor = "toy.transpose"(%tensor) {inplace = true} : (tensor<2x3xf64>) -> tensor<3x2xf64> loc("example/file/path":12:1)
```

在其中我们可以将其拆分为如下的部分

```
[out_ssa] = "[dialect].[op]"([operands]) {[attrs]} : ([type of operands]) -> [type of out_ssa] loc([path_to_source])
```

其中`[out_ssa]`是一个以SSA变量，`[dialect]`是上面提到的方言。

在这里面，开发者可以自己定义一套方言，并通过定义一组Transform将其转换为现有的其他方言或者转换到内置的方言，这使得MLIR的开发具有非常高的灵活性，开发者可以通过自由组合方言和Transform来设计一套完整的编译器。

# 前置知识

在开始设计之前，建议阅读编译器原理相关书籍或学习相关课程，了解一个编译器的基础结构和常见的设计方法。

## Build MLIR

MLIR作为了LLVM的一部分，通常情况下其都是直接在llvm-project下直接进行编译的，编译MLIR本体实际上非常简单。

首先自然需要克隆LLVM的仓库到你的目录下

```bash
git clone https://github.com/llvm/llvm-project
```

当然考虑到特色网络以及这个仓库的恐怖体积，通常还是加入`--depth=1`来减少克隆的数据量

接下来就是老三样，建立build目录，`cmake`生成编译指令，`cmake --build .`

```bash
mkdir build && cd build
cmake ../llvm -DCMAKE_BUILD_TYPE=Debug -DLLVM_ENABLE_PROJECTS="mlir" -DLLVM_BUILD_EXAMPLES=ON -DLLVM_ENABLE_RTTI=ON
```

上面指令的含意可以参考LLVM提供的文档[build LLVM with cmake](https://www.llvm.org/docs/CMake.html)，其中`-DLLVM_BUILD_EXAMPLES=ON`是为了让cmake同样会编译`mlir`提供的示例，之后会使用后续的示例，而`-DLLVM_ENABLE_RTTI=ON`则是为了编译具有LLVM风格运行时类型推断的二进制，从而减少后续可能会遇到的麻烦（LLVM的不少示例默认都是启用了RTTI的，但是编译LLVM默认却不会启用）。

然后就正式开始编译，进入漫长的等待

```bash
cmake --build .
```

## TableGen

TableGen是LLVM项目组开发的一套声明语言，这套语言最大的优点就是相对于直接使用C++定义可以减少不少的代码量，虽然实际上TableGen文件最后还是被生成到C++文件。

对于LLVM，其默认的TableGen具有多个对应不同应用场景的分支版本，这里我们需要用到的是`mlir-tblgen`，这个是专门用于生成开发MLIR所需要的定义文件的`tblgen`生成器。

