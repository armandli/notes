## C++17 Variant, Improved Union
to declare a variant:
```
std::variant<Class1, Class2> var; //where Class1 and Class2 does not need to be related in inheritance
```

to use the variant on common methods from both Class1 and Class2, do
```
struct CommonMethod {
  std::string operator()(const Class1& d){ return d.method(); }
  std::string operator()(const Class2& d){ return d.method(); }
};
std::visit(CommonMethod{}, var);
```

we can also use generic lambda to replace the CommonMethod if there is no strong distinction between Class1 and Class2 function call
```
auto CommonMethod = [](auto& d){ d.method(); }
std::visit(CommonMethod, var);
```

benefits of variant and visit:
* no semantics, no dynamic allocation
* easy to add new method, no need to change any implementation of class
* no need for base class, or to modify class hierarchy
* duck typing, allowing flexibility even if number of arguments could be different for different class, methods does not need to have the same signature
* potentially faster than dynamic dispatch

disadvantages:
* need to know all types at compile time
* might waste memory as std::variant has the size that is the max size of all types
* duck typing can also be disadvantage, depending on the rule that need to be enforced
* each operation need to have a separate visitor

in C++20, we can also use concept to enforce the duck typing on templates at compile time. by making a concept to capture what is required for each class for the visit. e.g. the following enfoces the return type of the method is a type convertible to string
```
template <typename T>
concept CommonMethodConcept = requires(const T v){
  {v.method()} -> std::convertible_to<std::string>;
};

auto CommonMethod = [](CommonMethodConcept auto& d) -> std::string {
  return d.method();
}
```

