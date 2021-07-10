### C++17 Class Template Argument Deduction
inability to do class template deduction is the reason all the `make_xxx` functions exist in c++11/14. e.g.

```
make_any
make_pair
make_tuple
make_optional
make_move_iterator
make_reverse_iterator
```

C++17 introduces class template argument deduction *CTAD*. it allows class template to be deduced from construction of the object. e.g.

```
template <typename T>
struct bar {
  bar(T){}
};

bar b1{100}; // deduced to int in C++17, error in C++11/14
```

C++17 introduces *deduction guide*, which is implicitly defined for every constructor of a class.

```
template <typename T> bar(T) -> bar<T>; // syntax of the deduction guide if written explicitly
```

CTAD leverages the existing FTAD rule.

#### Pitfall and Inconsistency of CTAD
FTAD supports partial deduction, CTAD does not
CTAD supports explicit deduction guides, FTAD does not
CTAD has special behavior for copies and moves

example:
```
template <typename T>
struct wrapper {
  wrapper(T){}
};
wrapper<int> w0{0};
wrapper<wrapper<int>> w1{w0};
wrapper w2{w0};
wrapper w3{w1};
```

w2 is a `wrapper<int>` because of the CTAD on copy constructor `template <typename T> wrapper(const wrapper<T>&) -> wrapper<T>;`
w3 is a `wrapper<wrapper<int>>`

implicitly defined deduction guides also have higher precedence than user defined guide.

this leads to inconsistency between make_xxx and CTAD

#### use case
scope_guard is great with CTAD
should refrain from using CTAD in template function bodies

#### Explicit Deduction Guide
users can create custom deduction guide to allow arbitrary types to be deduced. example use case:

```
list<int> s{0,1,2,3};
vector dst(s.begin(), s.end());
```

can be achieved using
```
template <typename InputIterator> vector(InputIterator first, InputIterator last) -> vector<iterator_traits_t<InputIterator>::value_type>;
```

but explicit deduction guide can be completely arbiturary and dangerous

can help with perfect forwarding

```
template <typename T>
struct wrapper {
  T data;
  template <typename U> wrapper(U&& x) : data{forward<U>(x)}{}
};

temlate <typename U> wrapper(U x) -> wrapper<U>;
```
