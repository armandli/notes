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
