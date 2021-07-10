### C++17 Fold Expression
in C++11/14 requires recursion and arbiturary expansion context such as initializer_list to unfold template variadic parameter packs, which is slow to compile and require arcane technique, or state and
mutability.

C++17 introduces fold expression that collapse a parameter pack into a single result using specified binary operator.

expression syntax:
```
( E op ...) becomes (E1 op ( ... op (En-1 op En)))
(... op E) becomes (((E1 op E2) op ...) op En)
(E op ... op I) becomes (E1 op (... op (En-1 op (En op I))))
(I op ... op E) becomes ((((I op E1) op E2) op ...) op En)
```

operation can be any of the 32 operators
I is the init expression, it does not contain an unexpanded parameter pack an does not contain an operator with precedence lower than cast at the top level, e.g. `(cout << ... << xs)`

precedence, associativity and sequencing order are given by the chosen operator, not by the parenthesis. e.g.

`(vec.push_back(std::forward<Ts>(xs)), ...);` is the same as `(... , vec.push_back(std::forward<Ts>(xs)))` because of the associativity of the comma

example usage:
```
template <typename T, typename ... Ts>
constexpr bool is_any_of(const T& x, const Ts&.. xs){
  return ((x == xs) || ...);
}
```

compile time unrolling example

```
template <auto N, typename F>
void repeat(F&& f){
  repeat_impl(f, std::make_index_sequence<N>{});
}

template <typename F, auto... Is>
void repeat_impl(F&& f, std::index_sequence<Is...>){
  (f(std::integral_constant<std::size_t, Is>{}), ...);
}
```

for tuple
```
template <typename F, typename Tuple>
void for_tuple(F&& f, Tuple&& tuple){
  std::apply([&f](auto&&... xs){
    (f(std::forward<decltype(xs)>(xs)), ...);
  }, std::forward<Tuple>(tuple));
}
```

iterating through types

```
template <typename T>
struct type_wrapper {
  using type = T;
};

template <typename ... Ts, typename F>
void for_types(F&& f){
  (f(type_wrapper<Ts>{}), ...);
}
```

check type list uniqueness

```
template <typename ...>
inline constexpr auto is_unique = std::true_type{};

template <typename T, typename... Rest>
inline constexpr auto is_unique<T, Rest...> = std::bool_constant<(!std::is_same_v<T, Rest> && ...) && is_unique<Rest...>>{};
```
