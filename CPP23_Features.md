## C++23 language features

### if consteval

```
#include <iostream>
 
constexpr int run(int i) {
    if consteval {
        return i*2;
    }
    else {
        return i;
    }
}
 
int main() {
    static_assert(run(10) == 20); // compile-time
    int a = 10;
    std::cout << run(a); // run-time
}
```

feature is equivalent to `if (std::is_constant_evaluated()) { }`

### reducing this

```
struct Pattern {
  template <typename Self> void foo(this Self&& self) { self.fooImpl(); }
};
struct MyClass : Pattern { void fooImpl() { ... } };
```

goal is to simplify code and allow for more control

### auto(x) and auto{x}

```
void pop_front_alike(Container auto& x) {
    using T = std::decay_t<decltype(x.front())>;
    std::erase(x.begin(), x.end(), T(x.front()));
}
// becomes:
std::erase(x.begin(), x.end(), auto(x.front()));

const std::string& str = "hello";
auto(str);  // Creates a new std::string (decayed copy)

int arr[] = {1, 2, 3};
auto(arr);  // Creates int* (arrays decay to pointers)
```

goal is replace decay_copy with langauge feature, allowing creating decay rvalue copy of input object

### extend init-statement to allow alias declaration

```
for (using T = int; T e : container) { ... }
```

### multidimensional subscript operator

```
#include <vector>
 
template <typename T>
class Array2D {
    std::vector<T> m;
    size_t w, h;

public:
    Array2D(size_t width, size_t height)
        : m(width * height), w(width), h(height) {}

    T& operator[](size_t i, size_t j) { return m[i + j * w]; }
};

int main() {
    Array2D<float> arr(4, 4);
    arr[1, 2] = 0.0f;
}
```

### static operator() and static operator[]

```
struct Fn {
    constexpr static int operator()(int x) {
        return x*10;
    }
};

int main() {
    static_assert(Fn::operator()(10) == 100);
    Fn x;
    static_assert(x(10) == 100);
}
```

### new lambda features

* attribute on lambdas
* () is more optional 
* call operator can be static
* deducing this allow for better recursion
* change scope of lambda return type

```
int main() {
    auto identity = [](int x) static { return x; };
    return identity(100);
}
```

static lambda

```
int main() {
    auto fib = [](this auto self, int n) {
       if (n < 2) return n;
       return self(n-1) + self(n-2);
    };
    static_assert(fib(7) == 13);
}
```

recursive lambda

```
nclude <iostream>
int main() {
    auto fn = [x = 0] mutable {
        return x++;
    };
    std::cout << fn() << fn();
}
```

optional ()

### [[assume]] new attribute

```
// Basic usage - compiler can optimize sqrt calculation
void process_positive(double x) {
    [[assume(x >= 0)]];
    return std::sqrt(x); // No need for negative number checks
}

// Loop optimization example
void process_array(int* arr, size_t size) {
    [[assume(size % 4 == 0)]];  // Assume size is multiple of 4
    [[assume(size > 0)]];       // Assume non-empty array
    for (size_t i = 0; i < size; i += 4) {
        // Compiler can optimize for 4-element chunks
        // No need for remainder handling
        arr[i] = arr[i] * 2;
        arr[i + 1] = arr[i + 1] * 2;
        arr[i + 2] = arr[i + 2] * 2;
        arr[i + 3] = arr[i + 3] * 2;
    }
}
```

### extend lifetime temporary in range based for

```
// Undefined Behavior in C++20 and earlier:
std::vector<std::vector<int>> getVector();
for (auto e : getVector()[0]) {  // temporary vector destroyed here!
    std::cout << e << '\n';      // accessing destroyed object
}

// Workaround required in C++20:
auto temp = getVector();
for (auto e : temp[0]) {
    std::cout << e << '\n';
}
```

```
// Now valid in C++23:
for (auto e : getVector()[0]) {  // temporary vector lives through the loop
    std::cout << e << '\n';
}

// More complex example that's now safe:
struct Matrix {
    auto getRow(int i) const { return /* ... */; }
};
std::vector<Matrix> matrices;

for (auto val : matrices.back().getRow(42)) {
    // Both the temporary from back() and getRow() are preserved
    process(val);
}
```

### new preprocessor directive

```
// Old style:
#ifdef _WIN32
    #define PLATFORM "Windows"
#else
    #ifdef __linux__
        #define PLATFORM "Linux"
    #else
        #ifdef __APPLE__
            #define PLATFORM "macOS"
        #endif
    #endif
#endif

// New style in C++23:
#ifdef _WIN32
    #define PLATFORM "Windows"
#elifdef __linux__
    #define PLATFORM "Linux"
#elifdef __APPLE__
    #define PLATFORM "macOS"
#else
    #warning "Unknown platform detected!"
#endif
```

### literal suffix for signed size_t

```
// Before C++23 - potential warnings or issues:
for (int i = 0; i < vec.size(); ++i)  // warning: comparison between signed/unsigned
for (size_t i = 0; i < vec.size(); ++i)  // verbose

// C++23 - clean and portable:
for (auto i = 0uz; i < vec.size(); ++i)  // perfect match with container's size_t
    std::cout << i << ": " << vec[i] << '\n';
```

### CTAD from inherited constructor

```
// Before C++23 - CTAD didn't work with inherited constructors
template<typename T>
struct Base {
    Base(T value) {}
};

template<typename T>
struct Derived : Base<T> {
    using Base<T>::Base;  // Inherit constructor
};

Derived d(42);  // Error in C++20: couldn't deduce T
                // OK in C++23: deduces Derived<int>
```

### simpler implicit move

```
// Before C++23 - inconsistent behavior
Widget&& foo(Widget&& w) {
    return w;  // Error in C++20
};
```
