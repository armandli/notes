### Structured Binding
simplify how to bind to multiple return types, or disassemble a tuple. e.g.
in C++11,

```
int a; char b; double c;
std::tie(a, b, c) = mytuple();
```

in C++17,

```
auto [a, i, b] = mytuple();
```

this is useful if there is multiple return values

```
auto [item, success] = mymap.insert(std::pair('a', 100));
```

can also be used in for loop

```
for (const auto& [key, value] : mymap){
  ~~~
}
```

structured binding allows multiple variable of different types to be initialized during for loop, which was not doable previously.

##### Structured Binding Concept
* structured binding introduces *names* for all the elements of the returned pair, not new variables
* it deduce types and allow usage of cv-qualification
* it supports structs, arrays, and custom types, and is fully customizable

format
```
/* qualifiers */ auto /* ref */ [/* identifier list */] = /* expr */;
```

at least one identifier has to be provided, the expression must be of array or class type

* because structured binding introduces names, the names are not copiable or movable, even if `const auto& [a, b] = expr;` the a and b are not taken by reference!
* can think of the names introduced as alias

#### customizing structured binding
1. must have valid

```
template <std::size_t I> T::get();
template <std::size_t I> get(T);
```

2. must specialize both

```
template <> struct std::tuple_size<T>;
template <std::size_t I> struct std::tuple_element<I, T>;
```

example:

```
struct io_channel {
  auto make_sender() { return sender{/*  */}; }
  auto make_receiver() { return receiver{/*  */}; }
};

template <std::size_t I>
auto get(io_channel& c){
  if constexpr (I == 0) { return c.make_sender(); }
  else                  { return c.make_receiver(); }
}

namespace std {
  template <>
  struct tuple_size<io_channel> : std::integral_constant<std::size_t, 2>{};

  template <>
  struct tuple_element<0, io_channel> { using type = io_channel::sender; };
  template <>
  struct tuple_element<1, io_channel> { using type = io.channel::receiver; };
}

int main(){
  io_channel channel;
  auto& [sender, receiver] = channel;
}
```

we need all those components because it's possible that the return type from the custom structured binding does not match exactly the member of the struct, and requires some type conversion

#### pitfall
* type deduction on structured binding names can be surprising

```
struct coordinate { int _x; int _y; };

coordinate c{0, 0};
auto& [x, y] = c;

x = 42;
assert(c._x == 42);

static_assert(std::is_same_v<decltype(x), int>); // but x is not a reference, but an int

```

same behavior is also noticable in decltype(auto)

* another consequence is that RVO/implicit move will not apply

```
struct person { std::string _name; int _age; };

std::string name_of_nth_person(const std::size_t idx){
  auto [nme, age] = global_person_registry[idx];
  return name;
}

auto example_name = name_of_nth_person(0); // string is copied out of name_of_nth_person
```

* any cv-qualifier applies to the hidden object, not the bindings themselves

```
std::tuple<char, int&> get_data();

const auto [c, i] = get_data();

c = 'a'; //compile time error, decltype(c) is const char

i = 42; // ok, is int&
```

also the types are retrieved via std::tuple_element

* structured binding cannot be nested

* structured binding cannot be captured in lambdas until C++20

* you cannot ignore a particular binding
