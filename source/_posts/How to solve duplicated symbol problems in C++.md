When compiling a program which sources sharing a same headers, for example, SWYAKL, there is possibility that error of duplicated symbol will appear during link stage.

For me, this problem is because SWYAKL is a header-only library, C++ compiler's preprocessor will expand headers to every source files include this header, so a function named `yakl::yakl_std_wrapper::round` had been added to top of every source files, and caused this error.

To solve this problem, the easiest way is change this function into an inline function.
```c++
// original version
namespace yakl {
namespace yakl_std_wrapper {
double round(double x); // In some condition, this will cause duplicated symbol error
}
}

// fixed version
namespace yakl{
namespace yakl_std_wrapper {
inline double round(double x); // This will not cause duplicated symbol error
}
}
```