## Lambda
lambda is a **closure object**. the type of lambda is a **closure type**, which is hidden by compiler and cannot be known. it is guaranteed that each lambda expression in code will always have a different closure type

lambda in C++ is equivalent in code to
```
class __closure_type {
public:
  RETURN_TYPE operator()(PARAMETERS) const { BODY }
private:
  ~~~ captured variables
};
```

labmda return type is deduced from lambda body, in C++14, deducibility of lambda body's return type is increased. e.g. in C++11 having multiple return statement would require explicit return type specification, but in C++14 that's not the case.

lambda can be captured and named using auto type deduction. it is the only way to capture a lambda as variable other than using `std::function<>`

closure may be copy initialized, but have no default constructor or copy assignment. e.g.

```
auto c3 = c1; //copy, ok
decltype(c1) c4; //error, no default constructor for closure type
c3 = c1; //error , no copy assignment operator
```

this is problematic with associative containers because they may take a comparator as object. if comparator is a lambda, it cannot default initialize the object, and require the object to be explicitly passed into the container. e.g.

```
auto comp = [](const T& a, const T& b){
  return a.v > b.v;
}
set<T, decltype(comp)> s; //error! comp cannot be default initialized as object in s
set<T, decltype(comp)> s(comp); //OK
```

in C++20 there will be support for default initialization for stateless lambdas.

lambda may not appear as unevaluated operand, such as in sizeof() or decltype() or alignas(). but you can use a variable to capture the lambda and pass it into a unevaluated operand.

```
auto fz = []{ return 12.3; }
cout << sizeof(fz) << endl;
```

lambda can access static names without passing it in as capture or as parameter. this includes both local static or global static members. for non-static members, such as local scope variable, class member, it needs to use capture.

notice the name used inside the lambda for captured variables is not the same variable as the variable captured from outside of the lambda. when a lambda captures a variable by copy, a member is created in the closure with a hidden name, which can only be referred to by the name of the variable outside of the lambda.

the data member in the closure is **direct-initialized**, just as if it would be in a member-initialization list.

capture by reference also creates a member variable inside the closure. the type of the name from within the lambda body is now a reference to the original variable from outside of the lambda.

there is default capture mode, which will capture every external variable if needed. default capture mode can be either capture by reference or capture by copy. but default capture mode is very dangerous and should not be used. you'll never know what you might be accidently captured when your intention is.

constness of closure means capture by copy variables cannot be modified, because the operator() in lambda is by default a const member. use `mutable` to avoid it

in C++17, there is **constexpr lambdas**

when lambda captures `this`, it is considered a member of the class it resides within. this means a lambda capturing this can access freely all members of the class. this is different from having a local class used as if it is a lambda.

the `this` captured by lambda is a reference to `this` outside of the lambda body. it is not a copy of `this`. class members are not captured when `this` is used

default capture mode captures `this`.

it is possible to capture the external object by value by using `[*this]`, which is available in C++17. in C++14, it is possible to do as **init-capture**. this is useful in case there is dangling reference from a reference capture.

note by default there is no such thing as `this` inside a lambda, `this` only exist if it is captured from external environment. this is different from function objects because fucntion objects always have a this.

lambda can capture variadic template parameter packs.

```
template <typename... Ts>
void foo(Ts&& ... args){
  auto lb = [args...](){
    goo(args...);
    joo(args...);
  };
}
```

stateless lambda can be implicitly converted to an pointer to function type.

```
class __closure_type {
public:
  constexpr operator __conv() const noexcept; //constexpr is added in C++17, not before
};
```

#### std::function
defined in `<functional>` header. it uses type erasure and can capture a lambda. however, type erasure is slow. however, `std::function` can capture any lambda so long as the return and arguments is convertible to the signature the `std::function` pointer type. e.g. a `function<double(int)>` can capture a lambda that takes a int and returns a int instead of a double, because a int is convertible to double

`std::function` is also memory expensive as it invovles allocating a virtual function table, a subclass object. function call is now indirect, cannot be optimized.

avoid recursive lambdas. the only way to do is to use `std::function` and it is very expensive. the reason a auto capture cannot be used is because the name during the declaration is not fully initialized in the body of the lambda, and therefore when the name is referred to in lambda the lambda could not initialize the body and therefore fails. `std::function` avoids this because `std::function` itself is fully defined, because of having a base class. its subclass is later defined once the lambda is complete.

use C++14 generic lambdas to solve the recursive problem.

```
auto fib = [auto n] constexpr noexcept -> int {
  return (n <= 2) ? 1 : fib(n - 1) + fib(n - 2);
};
```

the trick works by employing template's two phase translation to delay having to deduce the lambda's type until it is called. fib has to be static to avoid having to capture it, and that we have to explicitly specify the return type

choice between for loop and for_each, in C++17 there is a choice to use parallel algorithms for for loops. 

**use lambda has wrappers to allow inlining** because lambda type is guaranteed to be unique, this helps compiler in deducing to allow inlining of the lambda into its enclosing function. this means anything wrapped in a lambda has a higher chance of being inlined than comparing to not use a lambda wrapper. this includes when using a function pointer as comparator for a container, algorithm such as for_each, sort etc. e.g.

```
inline bool comp(const T& a, const T& b){
  ~~~
}
sort(begin, end, comp) //not inlined
sort(begin, end, ptr_fun(comp)); //not inlined
sort(begin, end, [](const T& a, const T& b){ return comp(a, b)}; ) //inlined
```

a stateless lambda is not zero sized because C++ requires the closure to occupy some space (1 byte). may want to use **empty base optimization (EBO)**.

in C++20, stateless lambda have default constructor and copy/move assignment operator.

there is problem with using unique_ptr with lambda, specifically in C++11, unique_ptr cannot be copied into lambda as a capture, and doing capture by reference on unique_ptr is bad. to use unique_ptr properly in lambda, you have to use init-capture.

#### Generic Lambda
template member basically means declaring templates as part of a class. there is no template argument deduction on class template. however in C++17 some of it is allowed.

member template is instead adding the template parameter to the member function of the class. this allows for template argument deduction. this is more flexible.

generic lambdas in C++14 allows for lambda to use template argument deduction to deduce its argument type.

generic lambdas also means there is no need to deduce complex type as part of lambda parameter.

problem with generic lambda is that its template argument type cannot be easily deduced inside the body of the lambda. we cannot easily use decltype to deduce the type of the parameter because of addition of references around the original type. we can however use `auto` to recover the type of the parameter, but won't help in all situations, such as knowing if the type contains pointer or reference in the first place. e.g.

```
auto wrapper = [](auto const* arg){
  auto temp = arg; //pointer and const still there
}
```

best and tedis solution:

```
auto wrapper = [](auto const * arg){
  remove_cv_t<remove_pointer_t<decltype(arg)>> temp;
//decay_t<decltype(arg)> temp; //also okay, decay_t perform the same type of transformation as pass by value
}
```

in C++20, we introduce template parameter back to address this issue:

```
auto wrapper = [] <typename T> (T const &arg){
  T temp = arg;
}
```

this would deal with the perfect forwarding issue too

#### Init-Capture
init capture captures the local variable by value and store a copy in the closure object, but now there is a way to refer to the member in the closure.

```
[amt = base](Stock const &stock){ /* use amt instead of base */ }
```

init capture can also do by reference

```
[&amt = base](Stock const &stock){ /* use amt instead of base */ }
```

but the init-capture can be more than just a local variable, it can be an expression. it is equivalent of writing `auto var = expr;`, can also do it using direct initialization as `[var(expr)]{}`

name declared in an init-capture is in the closure type's scope. however, the initializing expression is evaluated in the lambda's reaching scope. the enclosing scopes out to the inner most enclosing function. e.g.

```
[base = base * 1.02](const Stock& st) { ~~~ }
```

the second base is the base from the reaching scope, and the first base resides within the lambda. this is a bad use and a departure from most of the rest of C++ language, where a name comes into the scope before its initializer is evaluated; e.g. `int a = a;` would initialize a with itself, which is illegal

init-capture allows for move operation

```
auto f1 = [large_obj = move(large_obj)]{ ~~~ } //not doable without init-capture
```

this is especially useful for unique_ptr

```
auto ptr = make_unique<LO>();
auto f1 = [plo = move(ptr)]{ ~~~ };
```

init-capture also allows for calling functions to create a variable inside the lambda
```
auto p = [make_shape()]{ ~~~ } //error!
auto p = [shape = make_shape()]{ ~~~ } //OK
```

init-capture allows for capture of variables that does not have other reason to exist other than having to be captured by lambdas

lambda parameters can also be a variadic function parameter pack. a variadic lambda can be used as a **perfect forwarder**.
```
auto forward_lambda_var = [](auto && ... args){
  p(args...);
  q(forward<decltype(args)>(args)...);
};
```

#### Lambda vs Function Object
differences:
1. use of this in lambda body referrs to an enclosing class object, not the lambda's closure
2. do not know the declared names of captured items
3. lambdas have access to names in enclosing functions and enclosing lambdas through capture
4. lambdas may capture private and protected members of an enclosing class
5. lambda have no copy assignment
6. stateless lambda may be implicitly converted to pointer-to-function
