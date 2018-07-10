## R-value reference and perfect forwarding

lvalue can be converted to an rvalue in a context using **lvalue to rvalue conversion**, which basically means using the lvalue as if it is a rvalue.

lvalue can be **non-modifiable** if it is const, this is different from a rvalue. you can take the address of a non-modifiable lvalue, but you can't on a rvalue.

there are temporary rvalues that are class constructs. these can be created when a function returns a class object, or cast expression yielding a class type, but you still can't take the address of these class temporary object rvalues.

however, class object rvalue can be used to call its member functions. e.g.

```
cout << for_each(begin(v), end(v), Count()).count(); //where Count() is a temporary object and count() is a member function of Count object
```

a lvalue reference of type T can only be bound to a lvalue of type T without causing a type conversion in between, which may generate temporaries.

a lvalue reference to const T can bind to rvalue, even if the rvalue have type other than T. this is only valid if type conversion is possible. note this is a frequent source of error in code as conversions are overlooked.

note compiler is not obligated to reserve storage when binding a const lvalue reference to a lvalue.

const lvalue references allow rvalues to exist for interfaces, but does not allow efficient use of the rvalue as it is const and cannot modify the rvalue. the purpose of a rvalue reference is to allow binding to a rvalue while being able to modify the rvalue for efficiency purposes. this means const rvalue reference is useless.

a rvalue reference can only bind to rvalue. this is true even if it is a const rvalue reference.

binding of a rvalue reference to a rvalue increase the lifetime of the rvalue, just like const lvalue reference.

classes can have **move constructors** and **move assignment operators** that takes a rvalue reference of the class object. a class can have all of copy constructors, move constructors, move assignment operators and copy assignment operators. when a temporary is passed to the constructors of the class, move constructor takes precedence over copy constructor taking const lvalue reference when it comes to overloading resolution. the copy constructor is still legal for binding to lvalue and const lvalue as parameter.

typical signature of move constructor: `T(T&& x) noexcept;`. notice T here is not a template, and noexcept is true because move constructors should never throw.

a copy constructor typically performs a non-destructive copy, a move constructor usually performs a **destructive** copy. `auto_ptr` used to have a destructor copy constructor, and that was a problem.

with move constructor and move assignment operators, the use of `T(T& x);` copy constructors are pointless. it used to be an alternative to be able to steal resources from lvalue x, but this is no longer recommended because we expect lvalues to be alive after being copied.

you should declare both move constructor and move assignment operator for a class. this means the class imeplents **move semantics**. don't just do one and not the other.

**for types with no externally-managed resources, move is equal to copy and therefore does not need move semantics**.

for move assignment, the rvalue object that is moved from should remain in **moved-from state** after the move. the moved-from state should be
* destroyable
* copy-assignable and move-assignable
* must be empty of external resources, e.g. a container should be empty of elements

**the copy/move operations must agree on how an object resources should be managed**. this means move assignment operation should **not** have check on if it is copying from itself. and instead, allocate a new set of external resources and delete the old ones, then copy from the lvalue reference to the newly allocated external resources.

calling the move assignment operation on an object itself is perfectly legal. however, after such state, the object should be in the moved-from state, that is the object should be destroyable, copy assignable and move assignable, and is empty of external resources. this means move assignment operation should be implemented like this

```
String& String::operator=(String&& s) noexcept {
  //notice that there is no self check anymore
  char* temp = array;
  array = s.array;
  size = s.size;
  s.array = nullptr;
  s.size = 0;
  delete[] temp;
  return *this;
}
```

therefore move assignment of an object itself causes the object to explicitly enter moved-from state. this is important when the external resources are locks, as if the move assignment operation is implemented like swaps this would be wrong.

**generally the resource of a move should hold no resources after the move, even if the resource and destination are the same**

copy operations can throw exceptions due to the need to allocate external resources each time it is called. whereas move operations should **never** throw exceptions because it should never have allocation. therefore all move operations should be **noexcept**. move operations should be designed such that they don't throw.

the `std::move()` converts a lvalue into a rvalue. all names are considered lvalues, this includes rvalue references because they also have names.

e.g. new implementation of swap
```
template <typename T>
void swap(T& x, T& y){
  T temp(std::move(x));
  x = std::move(y);
  y = std::move(temp);
}
```

### noexcept
noexcept is a function that can compute whether the function would have exception or not. it can take a constant expression and evaluate it. e.g. swap implementation

```
template <typename T>
void swap(T& x, T& y) noexcept (
  std::is_nothrow_move_constructible<T>::value && std::is_nothrow_move_assignable<T>::value) {
  ~~~
}
```

noexcept for the swap is only true if those type traits are true. noexcept itself is the short notation for `noexcept (true)`

### return value optimization (RVO)
compile is allowed to eliminate the returned local variable, its copy and destruction, and substitute the return location. however, RVO may not always be applied.

in order to allow RVO, **prefer initialization over assignment**, this means we should use the constructors more and avoid the assignment operations.

because return by value returns a non-const rvalue, this could be misused to call member functions on that temporary. declaring const on the returned rvalue is **detrimental** and should be avoided. it used to be a recommended practice, but not anymore. this is because declaring return value const nullifies the move assignment operation because move assignment operation is always non-const. when the returned type is const-qualified, overload resolution will select for copy constructor taking const lvalue reference over the rvalue reference move constructor.

**should not use `std::move(ret);` for functions return by value.** it is not necessary, even counter-productive. this is because returned type is already a rvalue, std::move will convert it into a rvalue reference, which disables the RVO because RVO operates on rvalues, not rvalue references.

RVO is often implemented in the front end because it's langauge specific. compilers will consider the following optimization strategies in order:
1. RVO
2. move constructing the return value
3. copy constructing the return value

RVO does not apply to return a value parameter. i.e. the returned value is a function lvalue reference parameter. however, don't try to use std::move to solve this issue as compiler will still attempt to do RVO.

**only use std::move() to deal with return a function parameter if it is a rvalue reference**. e.g.

```
T operator+(T&& a, const T& b){
  return a += b; //no RVO, copy result
}
```

change this to

```
T operator+(T&& a, const T& b){
  return std::move(a += b); //no RVO, move result
}
```

**you can only do std::move() on an object once!**

### value category
* lvalue designates a function or an object
* xvalue designate an expiring value, refers to an object that's typically near the end of its lifetime. expressions involving rvalue references typically yields xvalue. note that an rvalue reference to a function is not an xvalue!
* prvalue is a pure rvalue, an rvalue that's not an xvalue. such as nullptr, or literal values
* glvalue is generalized lvalue, containing both lvalue and xvalue. it's an expression that refers to an object in memory or to a function
* rvalue contains both xvalue and prvalue, think of it as expression that can be used to initialize an rvalue reference.

### Reference collapsing
dealing with situation where a multiple reference modifier are applied to a type. collapsing rule is any existence of & becomes lvalue reference, && and && maintains rvalue reference.

type deduction rule says
* when it's an lvalue of type T, it is deduced into T&
* when it's an rvalue of type T, it is deduced into T

this means an auto type deduction could produce a lvalue reference when needed. same rule also applies to template argument deduction.

function overloading permits all of:
```
template <typename T> void foo(T& p);
template <typename T> void foo(T&& p);
template <typename T> void foo(const T& p);
```

when all overload exists, `T&` will bind for lvalue reference parameters, `const T&` will bind to const lvalue reference parameters,  `T&&` will bind to rvalue parameters

the `T&&` is a forwarding reference and given context where there is no overloads such as `T&` or `const T&`, it can bind to anything. in a lvalue reference context, T is deduced into `T&` and is bound, in const lvalue reference context, `T` is deduced into `T`. generally it is a bad idea to overload `T&&` in a type deduction context.

in STL, overloading is used for functions that take forwarding references, this is for optimization purposes.

emplacement member functions in STL containers are used to construct element within the container, avoiding any temporary creation, moving or copying. emplaced elements are often constructed using placement new. the contaienr need to have a different name compared to insertion due to template argument deduction needs.

`std::forward` is used to transfer forwarding references, perserving its value type perfectly to the next call. while `std::move` unambiguously cast to rvalue reference.

a const T&& is not a forwarding reference. a forwarding reference cannot have the explicit const modifier.

following T is not a forwarding reference:
```
template <typename T>
struct X {
  void op(const T&);
  void op(T&&);      //not a forwarding reference because T is already deduced.
};
```

prefer not to overload function with forwarding references because some overload resolution can be very surprising. e.g.

```
template <typename T> struct X {
  void op(const T&);                  //lvalue version
  void op(T&&);                       //rvalue version
  template <typename S> void op(S&&); //universal version
};

X<T> a; a.op(T()); //call rvalue version
const T b; b.op(b); //call lvalue version
T c;
a.op(c); //call universal version
```

`std::decay` help deduce the value type, can be used to limit the greediness of forwarding reference. e.g.

```
template <typename S, typename T>
using similar = is_same<decay_t<S>, decay_t<T>>;

template <typename T> struct X {
  void op(const T&);
  void op(T&&);
  template <typename S, typename = enable_if_t<!similar<S, T>::value>>
  void op(S&&);
};

X<T> a; a.op(T()); //call rvalue version
const T b; b.operation(b); //call lvalue version
T c; c.op(b); // call lvalue version instead of call universal version
T* pt;
a.op(pt); //call universal version
```

however, the better solution is to not use overloading in this senario.

should only use the forwarding reference one in the context because it could be moved.

implementation of `std::forward`

```
template <typename T>
constexpr T&&
forward(typename remove_reference<T>::type& t) noexcept {
  return static_cast<T&&>(t);
}
template <typename T>
constexpr T&&
forward(typename remove_reference<T>::type&& t) noexcept {
  return static_cast<t&&>(t);
}
```

implemented as overloaded function template.
