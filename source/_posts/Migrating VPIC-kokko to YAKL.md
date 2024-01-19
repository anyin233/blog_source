---
title: Migrating VPIC-kokko to YAKL
date: 2024-01-19 12:00:00
tags: [YAKL,Kokkos,HPC]
category: [Tech,HPC,YAKL]
---



# Why I want to migrate it to YAKL

YAKL is a light-weight performance portability library, and it is a header-only library. It supports many main-stream HPC platform. To compare with Kokkos, YAKL is compatible with typical CMake project, and is also easy to migrate to new platform.  
Currently I want VPIC can run on a new platform, which is different with todays popular platform like CUDA, HIP etc. But first Kokkos depends on gcc 8.4.0+, which haven't been implemented on this platform and in the same time I want use a library with few feature so that I can implement new API without any restriction, so that I choose YAKL.

# What can be replaced directly using YAKL
```
Kokkos::View         -> yakl::Array
Kokkos::parallel_for -> yakl::parallel_for
....
```

# What should I implement 
```
Kokkos::ScatterView
Kokkos::parallel_reduce
Kokkos::TeamPolicy (Maybe replaced by yakl::parallel_outer and yakl::parallel_inner)
Kokkos::HostMirror (Maybe replaced by yakl::Array with memHost)
```