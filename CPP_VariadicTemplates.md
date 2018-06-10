## Variadic Templates
a **parameter pack** is a template parameter that accepts 0 or more template arguments.
a template that has a parameter pack is a **variadic template**.

every time a template parameter pack is used, we are assumed to expand the pack at the location as if it is a pattern expansion. e.g.

```
template <typename... Ts> //template parameter pack
class C {
  tuple<const Ts *...> tts; //expand the parameter pack Ts, which each template receives a const and * modifier
};
```

if we want to use the template parameter packs in a function as function parameter types, we expand it into a function parameter pack
```
template <typename... Ts> //template parameter pack
void foo(Ts... args){ // function parameter pack
  print(args...); //function parameter pack expansion, as a pattern
}
```

forwarding tempalte parameter packs is also considered a pattern expansion

```
template <typename... Ts>
void process(Ts&& ... args){
  do(forward<Ts>(args)...); //also considered a expansion using pattern forward<Ts>(args), expanding on the Ts and args
}
```

variadic templates avoid the issue of having to write a combinatorics number of function templates just to handle different types and number of types.

variadic templates allow emplace_backs. push_backs may use copy constructor or move constructor on existing object, emplace_back allows to construct an object right on the memory inside of a container without creating any temporary.

contexts of pack exapansion:
* templates
* function parameters
* function arguments
* base class list
* member initialization list
* throw, but throw itself will be deprecated
* lambda capture

C++17 added fold expressions on parameter pack expansion pattern, allowing for left or right unary or binary folding, using patterns such as
```
template <typename ... Ts>
auto sum(Ts... args){
  return (args + ... + 0);
}
```

base class expansion also considers the public/private specifier part of the pattern. this means although we cannot create a class without any base class directly in C++ syntax:
```
struct S : { //illegal
  S(): {}    //illegal
  int alignas() mem; //missing argument
};
```

we can do this with template parameter packs expansion to no base class and consider it valid.

but there is limit to template parameter packs

```
template <size_t... idxes, typename... Ts> class Error; //error, can only have 1 template parameter packs here, because it's for class
```

it is possible for a template to have multiple template parameter packs for functions. This is useful when doing ziping of multiple packs inside a function. However, explicitly specifying each template for the template parameter packs will not work because it always assumes the first template parameter pack takes all the template arguments, and leave the second to have nothing. the only way to use multiple template parameter packs is to use template argument deduction on the deduce the 2 packs, or use template argument deduction to deduce the second or following packs while leaving the frst explicit.

```
template <typename... T1, typename... T2>
tuple<T1..., T2...>
paste(tuple<T1...> const &a, tuple<T2...> const &b){
  return tuple<T1..., T2...>();
}

auto t1 = make_tuple(1.2f, 1L);
auto t2 = make_tuple('a', 1);
auto t = paste<float, long long, wchar_t, short>(t1, t2); //error, first pack contains 4 items, second pack contains none
auto t = paste(t1, t2); //works
```

pack expansion could fail if t1 and t2 does not have the same length; this is only a failure if multiple packs are expanded at the same pattern, but if they are expanded on different patterns, it is fine

using index sequence in pattern expansion is a common and useful implementation technique. C++14 facilitate generate sequential sequences of indicies;

```
template <size_t... idx>
using index_sequence = ~~~; // an arbitrary sequence

template <size_t... Is1, typename... Ts1, size_t... Is2, typename... Ts2>
tuple<Ts1..., Ts2...> paste_helper(index_sequence<Is1...>, tuple<Ts1...> const &a, index_sequence<Is2...>, tuple<Ts2...> const &b){
  return tuple<Ts1..., Ts2...>(get<Is1>(a)..., get<Is2>(b)...);
}
~~~
auto t = paste_helper(index_sequence<0, 1>(), t1, index_sequence<0, 1, 2, 3>(), t2);
```

the index_sequence type is useful for capturing non-type template parameter pack and use template argument deduction to deduce the pack without having to explicitly expand them, this is especially useful in a context where multiple template parameter packs are involved in a function signature.

variadic template can be placed in `sizeof...()` to get the length of the pack

variadic templates are often used with the first/rest manipulation pattern. this pattern often involves a undefined generic function signature, then followed by 2 specializations, one on a empty parameter pack as the base case, and the inductive care takes in a template parameter pack with a realized template

```
template <typename... Ts> struct Count;
template <> struct Count <> { 
  enum : size_t { c = 0 };
};
template <typename T, typename... Ts> struct Count<T, Ts...> {
  enum : size_t { c = 1 + Count<Ts...>::c };
}
```

the variadic argument pack does not have a order of evaluation and require either C++17 fold expression to realize the order, or use initializer_list.e.g.

```
template <typename... Ts>
void print(Ts... args){
  initializer_list<bool>{ (cout << args << endsl, true)... }; 
}
```

C++17 fold expression can be used to replace first/rest usage.

```
template <typename T>
auto sum(T arg){
  return arg;
}

template <typename T, typename... Ts>
auto sum(T arg, Ts... args){
  return arg + sum(args...); // unary right folding, expands to a1 + (a2 + (a3 + a4))
}
```

in order to handle empty packs with folding, need special cases

```
template <typename T> auto sum(T arg){
  return arg;
}
auto sum(){ return 0; }
template <typename T, typename... Ts>
auto sum(T arg, Ts... args){
  return arg + sum(args...);
}
```

binary folding

```
template <typename... Ts>
auto sum(Ts... args){
  return (args + ... + 0); // binary right fold, expand to a1 + (a2 + (a3 + (a4 + 0)))
}
```

alternatively use constexpr if

```
template <typename... Ts>
auto sumfold(Ts... args){
  if constexpr(sizeof...(Ts) > 0) return (args + ...);
  else                            return 0;
}
```

iostream is best working with left folding, while assignments best work with right folding.

```
template <typename... Ts>
void printer(Ts && ... args){
  (std::cout << ... << args) << '\n';
}

template <typename... Ts>
void assignall(T && source, Ts && ... args){
  (args = ... = source);
}
```

count can be shrunk down using fold expression

```
template <typename... Ts>
struct Count {
  enum : size_t { value = (!!sizeof(Ts) + ... + 0) };
};
```

there is unary right fold for operators && and ||, which will supply a default argument for an empty pack.

##### literal operator template
takes a template parameter pack instead of an argument. the parameter pack must be exactly char...

```
template <char... chars>
constexpr unsigned operator "" _len() noexcept {
  return sizeof...(chars);
}
```

this signature is fixed, adding additional template anywhere will cause error

##### Variable Template
in C++14, a variable can be taking a template too
```
template <typename T>
constexpr auto happiness_v = And<T,
                                 is_polymorphic,
                                 is_nothrow_copy_constructible,
                                 is_transparent>::value;

static_assert(happiness_v<T>, "unhappy i am");
```

and with C++17 folding

```
template <typename T, typename <typename> class... Preds>
constexpr auto and_v = (Preds<T>::value && ... );
```
