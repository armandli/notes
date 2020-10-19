## C++17 Variant, Improved Union
to declare a variant:
```
std::variant<Class1, Class2> var; //where Class1 and Class2 does not need to be related in inheritance
```

to initialize, use brace init or copy assignment
```
std::variant<int, float> var1{ 1.0F };
std::variant<int, float> var2 = 1.0F;
```

if variant types are similar, such as float and double such that compiler cannot deduce, use `std::in_place_index<>`
```
std::variant<float, double> var1{ std::in_place_index<0>, 1.2 };
```

if a type in std::variant does not have a default constructor and thus no initialization for the std::variant, use `std::monostate` to represetn empty type.

to get the value if you know the type
```
std::get<float>(var1); //throws std::bad_variant_access if get fails
```

to try to get a type from vriant, use `std::get_if` or `std::holds_alternative`
```
if (const auto ptr(std::get_if<float>(&var1)); ptr){
  //ptr is valid
}
if (std::holds_alternative<int>(var1)){
  //true
}
```

std::variant object lifetime is safe, when type is replaced in the variant variable, the destructor is called for the old type

be careful of type conversion. e.g.
```
std::variant<std::string, int> var1;
var1 = "hello world"; // problem, "hello world" has type char*, not string!
```
this problem is fixed in C++20

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
