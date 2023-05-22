---
title: MPI_Cart_Create 使用方法
date: 2022-12-25 22:05:10
tags: MPI,并行编程
---

在使用MPI的过程中可能会涉及到一些针对特定的拓扑关系的模拟情况，如**FOX算法求矩阵乘积**。此时需要涉及到如何简便地模拟出特定的方阵以及元素之间的关系。MPI提供了一套简便地API可以快速地创建出携带所需要的拓扑关系的`MPI_Comm`。

<!--more-->

# MPI Process Topology Functions

该系列的函数即是MPI中用于处理相关的拓扑关系的相关工具。常用的包括了`MPI_Cart_create`等方法，具体的API可以参考[MSDN](https://learn.microsoft.com/en-us/message-passing-interface/mpi-process-topology-functions)。其中较为常用的包括`MPI_Cart_create`、`MPI_Dims_create`等方法。

## `MPI_Cart_create`

该方法API如下所示

```c
int MPIAPI MPI_Cart_create(
        MPI_Comm              comm_old,
        int                   ndims,
        _In_count_(ndims) int *dims,
        _In_count_(ndims) int *periods,
        int                   reorder,
  _Out_ MPI_Comm              *comm_cart
);
```

其中`ndims`表示维度数量，`dims`表示全局的各个维度的大小，`periods`表示（不知道），`reorder`表示是否按照划分的小块重新计算rank值。

使用该方法后产生的`MPI_Comm`将会携带所需要的维度信息，例如

```c
MPI_Cart_create(MPI_COMM_WORLD, 2, dims, periods, false, &cart_comm);
```

执行后，`cart_comm`携带的维度信息为全局为一个`dims[0] * dims[1]`的矩阵。

## `MPI_Dims_create`

> 未使用过，故不确定其真实效果

# 使用案例

Fox算法求矩阵乘积

```c
#include <iostream>
#include <mpi.h>
#include <cmath>
#include <fstream>

using namespace std;

#define RIDX(i, j, dim) (i * dim + j)

const int N = 256;

int main() {
    MPI_Init(NULL, NULL);

    int rank_num, world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank_num);

    if (pow(int(sqrt(world_size)), 2) != world_size) 
        printf("Wrong world size\n");


    int proc_sqrt = floor(sqrt(static_cast<double>(world_size)));
    int n = N / proc_sqrt; // N is big matrix size
    int n_sqrt = n * n; // small matrix size (n * n)

    /* check for pragmas */
    if (world_size < 4)
    {
        printf("This algorithm requires at least 4 processors\n");
        MPI_Finalize();
        return 0;
    }
    if (proc_sqrt * proc_sqrt != world_size)
    {
        printf("processor count must be square.\n");
        MPI_Finalize();
        return 0;
    }
    if (N % proc_sqrt !=0) {
        printf("N mod procs_sqrt !=0  ");
        MPI_Finalize();
        return 0;
    }

    if (rank_num == 0)
    {
        printf("Computing %d * %d matrix, submatrix size is %d * %d\n", N, N, n, n);
    }

    /* create matrixs */
    int *A = new int[n_sqrt];
    int *B = new int[n_sqrt];
    int *C = new int[n_sqrt];
    int *T = new int[n_sqrt];

    for (int i = 0; i < n; ++i)
    {
        for (int j = 0; j < n; ++j)
        {
            A[RIDX(i, j, n)] = (i + j) * rank_num;
            B[RIDX(i, j, n)] = (i + j) * rank_num;
            C[RIDX(i, j, n)] = 0;
        }
    }

    /* split comm */
    MPI_Comm cart_comm, cart_col, cart_row;
    int all_rank, col_rank, row_rank;
    int cart_coords[2];
    int dims[2], periods[2];
    dims[0] = dims[1] = proc_sqrt;
    periods[0] = periods[1] = true;
    /* create global node matrix */
    MPI_Cart_create(MPI_COMM_WORLD, 2, dims, periods, false, &cart_comm);
    MPI_Comm_rank(cart_comm, &all_rank);
    MPI_Cart_coords(cart_comm, all_rank, 2, cart_coords);
    /* split comm via col & row */
    MPI_Comm_split(cart_comm, cart_coords[0], cart_coords[1], &cart_row);
    MPI_Comm_split(cart_comm, cart_coords[1], cart_coords[0], &cart_col);
    MPI_Comm_rank(cart_row, &row_rank);
    MPI_Comm_rank(cart_col, &col_rank);


    MPI_Request req_send, req_recv;
    MPI_Status status;
    for (int i = 0; i < proc_sqrt; ++i)
    {
        /* swap rows of B */
        MPI_Isend(B, n_sqrt, MPI_INT, (cart_coords[0] - 1 + proc_sqrt) % proc_sqrt, 1, cart_col, &req_send);
        
        int broader = (i + cart_coords[0]) % proc_sqrt;
        if (broader == cart_coords[1]) std::copy(A, A + n_sqrt, T);
        /* boardcast A */
        MPI_Bcast(T, n_sqrt, MPI_INT, broader, cart_row);

        /* local mul */
        for (int r = 0; r < n; ++r)
        {
            for (int c = 0; c < n; ++c)
            {
                for (int k = 0; k < n; ++k)
                {
                    C[RIDX(r, c, n)] = T[RIDX(r, k, n)] * B[RIDX(k, c, n)];
                }
            }
        }

        MPI_Wait(&req_send, &status);
        /* finish row swap */
        MPI_Recv(T, n_sqrt, MPI_INT, (cart_coords[0] + 1) % proc_sqrt, 1, cart_col, &status);
        std::copy(T, T + n_sqrt, B);
    }

    /* gather global matrix */
    int *matrixA = new int[N * N];
    int *matrixB = new int[N * N];
    int *matrixC = new int[N * N];

    MPI_Gather(A, n_sqrt, MPI_INT, matrixA, n_sqrt, MPI_INT, 0, MPI_COMM_WORLD);
    MPI_Gather(B, n_sqrt, MPI_INT, matrixB, n_sqrt, MPI_INT, 0, MPI_COMM_WORLD);
    MPI_Gather(C, n_sqrt, MPI_INT, matrixC, n_sqrt, MPI_INT, 0, MPI_COMM_WORLD);

    if (rank_num == 0)
    {

        ofstream Af("c.data/a.txt"), Bf("c.data/b.txt"), Cf("c.data/c.txt");
        for (int i = 0; i < N; ++i)
        {
            for (int j = 0; j < N; ++j)
            {
                Af << matrixA[RIDX(i, j, N)] << "\t";
            }
            Af << "\n";
        }
        
        for (int i = 0; i < N; ++i)
        {
            for (int j = 0; j < N; ++j)
            {
                Bf << matrixB[RIDX(i, j, N)] << "\t";
            }
            Bf << "\n";
        }

        for (int i = 0; i < N; ++i)
        {
            for (int j = 0; j < N; ++j)
            {
                Cf << C[RIDX(i, j, N)] << "\t";
            }
            Cf << "\n";
        }

        Af.close(), Bf.close(), Cf.close();
    }


    /* free resources */
    MPI_Comm_free(&cart_comm);
    MPI_Comm_free(&cart_col);
    MPI_Comm_free(&cart_row);
    delete[] A;
    delete[] B;
    delete[] C;
    delete[] T;
    delete[] matrixA;
    delete[] matrixB;
    delete[] matrixC;
    MPI_Finalize();
}
```

