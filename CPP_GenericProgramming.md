## Generic Programming

### Enumeration
* enum values are constants, explicit value assigned to enum are expected to be constant expression, capable of compile time evaluation
* problem: underlying type for enum in C++03 was never known (implementation defined), but in C++11 the underlying type can be explicitly specified
* problem: implicit conversions on enum can be applied but is unknown. e.g. `if (et > medium)` where both et and medium is enum value, both are implicitly converted to int for comparison but that's not a guarantee, could also be unsigned int
* problem: because underlying type cannot be standardized, memory layout is different on different platforms for having enum member in class.
* problem: enum values can have conflict with other enum values of another enum type

#### Scoped Enumeration
```
enum class e {
  e1,
  e2,
};
```

old enum are called **unscoped enumeration**

scoped enum requires explicit operator for type conversion, scoped enum value cannot be accessed without enum type scope.

#### Underlying Type Specifier
```
enum k : unsigned char {};
```

Suggested Usage:
* use enum class for enum in namespace scope
* use unscoped enum in class definition, or more restricted scopes

Obtain underlying type of enum: `std::underlying_type<enum_type>::type` in `<type_traits>` header

### Constant Expression
a **constant expression** is a restricted form of expression in which every operand has a value that can be determined at translation time, and no operator modifies an operand
most common form of constant expression is a **integral constant expression**

certain expression must be constant:
* case label expression
* bit-field length
* initializer in enumeration definition
* dimension in array declarator
* initializer for static data member initialized within its class definition
* template non-type argument of integral or unscoped enumeration type

In C++03 the following are constant expression:
* literals
* enumerators
* sizeof
* const-qualified objects (include static data members) of integral or enumeration types initialized with constant expressions
* non-type template parameters of integral or enumeration types

e.g.
```
int x[sizeof(f(0))]; //sizeof is a unevaluated context, obtains the sizeof return type for f(0)
```

problem with C++03 `<limits>` header, `numeric_limits<T>::max()` does not return a constant value. C++11 fixed this by **constexpr**

#### Constexpr
types:
* constexpr function
* constexpr object
* constexpr constructor
* constexpr member function

##### constexpr function
* very restricted in C++11, but C++14 allowed for more expression to be usable in constexpr function
* calling a constexpr function may or may not result in a constant expression, e.g. calling constexpr function with non-constant parameter will result in runtime evaluation of the constexpr function
* C++11 does not allow void as return type, C++14 allows for void as return type
* C++11 does not allow modification to variables within constexpr function, C++14 allows
* constexpr function assumed to be inlined
* C++14 allows to use multiple statements in constexpr function, allowing if, while, for
* recursion depth of constexpr function could be around 512 or 1024
* evaluation resulting in arithmetic error could cause compilation failure, e.g. divide by 0

example:
```
constexpr size_t pow10(size_t digits){
  return (digits == 0) ? 1 : 10 * pow10(digits - 1);
}
```

##### constexpr object
constexpr objects are guaranteed to be const. assign a constexpr expression to a constexpr object to make sure that the result of the expression is actually const.

following may or may not be a compile time constant
```
int const p6 = pow10(six); //even if pow10 is constexpr function and six is const, p6 may still not be const
```

##### Static vs. Dynamic Initialization
static initialization occurs before program startup.

to statically initialize an object in C, define the object with static storage duration, either with `extern` or `static`, or place the definition at file scope. if the initializer isn't a constant expression, the compiler will let you know.

in C++, to use static storage duration, use the keyword `extern`, `static`, or `thread_local` or place the definition at namespace scope. however, if initializer is not constant expression, compiler will not tell you and use dynamic initialization. C++ constructor is definitely not usable to guarantee static initialization.

##### constexpr constructor
guarantees static initialization for literal class types

C++11 constexpr constructor must not have any executable statements in its body, and must initialize every non-static data member with a constant expression

constexpr constructor in C++14 allow executable statements in constructor body.

constexpr constructor can still not be static if member initialization expression contains non-static expression. must use **constexpr object definition** to ensure static initialization.

use constexpr lavishly

##### constexpr member function
C++11 constexpr member functions are implicitly const, compiler may warn if a constexpr member function is explicitly declared const

C++14 constexpr member functions are not implicitly const, and therefore need the const specifier

##### Type Sinks
type sinks are bad, they can take any type that may not have valid meaning to the object for initialization. e.g. Date class

```
class Date {
public:
  constexpr Date(size_t m, size_t d, size_t y) noexcept: month(m), day(d), year(y) {}
};

constexpr Date crasher(24, 10, 1929);
```

to avoid type sinks, use distinct type for each concept in Date, such as a Day type, a Month type, and a Year type. this way day and month could avoid the ordering error in constructor of Date.

##### Most Vexing Type
the above suggestion would use constructors in the creation of a Date object, which could cause most vexing type error. to avoid most vexing type error, use **user defined literals** of the Day, Month, Year object.

##### user defined literal and literal operator
```
operator "" identifer
```

e.g.

```
constexpr Day operator "" _day(unsigned long long d) noexcept { //where _day will be used as ud-suffix, has to be unsigned long long, not size_t, because ull is builtin
  return Day(static_cast<size_t>(d));
}
```

most of the time we expect user defined literal to be constexpr, a compile time constant, so need to add `constexpr` and `noexcept` into the literal operator.

Date creation becomes `Date d{1_day, 1_month, 1976_year};`

but how far can we go with user defined literal ? should we have udl for Date itself ? like 1929'10'24_date ?

udl comes in several varieties: integer, floating, character and string. types of the literal operator arguments are fixed to long double for floating, unsigned long long for integer, char types for char, and string for **string literals**, where string type is a const char*

```
T operator "" _str(std::string, size_t);
```

there is raw literal operator `T operator "" _rrr(char const*)` where is selected if there is no better match to an integer or floating literal. e.g. you can use user defined literals to define alternative based numbers such as base5 numbers using raw literal operator to scan the entire integer or floating point value as if it is a string and generate the base5 value for it.

there is also variadic character template.

constexpr is better than const, constexpr is better than inline.
