### C++17 Constructor template argument deduction
C++17 allows template argument deduction on the constructor for types now. this means we no longer need to be forced to type out the class name along with all of its required template types when we construct an object. e.g.

```
vector v = {1,2,3};
```

notice the deduction will still need arguments in order to know how to deduce the type for vector.

this may be replacing make_pair functions:

```
std::pair p{1, "13"}
```

as that is easiser than:

```
auto p = std::make_pair(1, "13");
std::pair<int, std::string> p2{1, "13"};
```
