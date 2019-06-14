## Record of known hand optimizations needed for GCC in order to generate optimized code
### Tail Recursion
GCC does tail recursion optimization, transform a tail recursed function call into a jump instead of call instruction. but somtimes this optimization fails. one of
the reason could be stack unwinding needed to be done that GCC cannot prove to remove, disabling tail recursion.

code that allows for tail recursion optimization:
```
extern int tail(int);
int f(int a){
  return tail(a + 1);
}
```

code that does not allow tail recursion optimization:
```
extern int tail(int);
int f(int a){
  return tail(a + 1);
}
```

tail recursion that could not be optimized:
```
#include <string>
using namespace std::string_literals;
extern int tail(int);
extern void out(std::string&);
int f(int a){
  if (a > 10){
    std::string s = "a = "s + std::to_string(a); //need to unwind the stackframe here, disables tail recursion, 
    out(s);
  }
  return tail(a + 1);
}
```

to fix this:
```
#include <string>
using namespace std::string_literals;
extern tail(int);
extern void out(std::string&);
namespace {
  [[qnu::noinline]] void inner(int a) noexcept {
    auto s = "a = "s + std:::to_string(a);
    out(s);
  }
}
int f(int a){
  if (a > 10){
    inner(a);
  }
  return tail(a + 1);
}
```

### assumptions
somtimes compiler cannot prove by value range propagation that some value could not happen and need to generate code for the special case. if the user knows exactly that would never happen, we can add `__builtin_unreachable()` so compiler correct its assumption on value range propagation and does not generate the additional conditional branch to return 42 here

```
static inline int f(int a){
  switch (a){
  case 0:
    return 0;
  case 1: case 2: case 3:
    return 1;
  case 4: case 5: case 6: case 7: case 8: case 9: case 10:
    return -1;
  default:
    return 42;
  }
}

int q(int a){
  if (a < 0 || a > 3)
    //tells compiler to eliminate handling of other possibilities in the f, with this, function f returns value a or -1, which is much
    __builtin_unreachable();
    shorter code
  return f(a);
}
```

### Vectorization
compiler in a lot of cases cannot prove to use vectorized instructions. for example, you iterate to add each element of 2 vector of int together. it can be
optimized to use vectorized int add instruction, but if the vector are of different size, then you will still need to add the uneven sized vector together. compiler
does this by doing a runtime check on the size of the 2 vectors, if proven to be the same length, use the vectorized instructions, otherwise it uses the slower
element by element loop

vectorized loop with runtime check to use slow loop if condition for vectorization is invalid
```
#include <stddef>
#ifndef R
#define R __restrict
#endif
using float_type = double;
void f(size_t n, float_type* R res, const float_Type* R a, float_type b){
#ifdef P
#pragma GCC ivdep
#elif defined OMP
#pragma omp simd
#endif
for (size_t i = 0; i < n; ++i)
  res[i] = a[i] * b - 1.0;
}
```

on linux, there are parallelized version of sqrt, sin, cos etc, and vectorization is going to be able enable calling these instructions (not library), however, error handling in the result of sqrt or sin could break the optimizer to choose not to use those instructions.

sometimes vectorization could not be done due to alignment of the array, we have to manually insert vectorization alignment instruction
```
void foo(int *p, int *q, int *r){
#ifdef DECLARE_ALIGN
  p = __builtin_assume_aligned(p, 16);
  q = __builtin_assume_aligned(q, 16);
  r = __builtin_assume_aligned(r, 16);
#endif
  for (int i = 0; i < 1024; ++i)
    p[i] = q[i] + r[i];
}
```

#### Micro Optimization in C
when declaring prototype, use f(void) instead of f() to save 2 byte of instruction, or use `-Wstrict-prototypes`
