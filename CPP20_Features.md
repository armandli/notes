### constexpr expansion
* virtual constexpr functions are now allowed
* allowing try/catch inside constexpr
* allow changing members inside of a union
* stl string and vector uses constexpr more
* std::is_constant_evaluated() to check if expression is constexpr
* declaring function consteval will cause error if const expression is not evaluated at run time
* constinit keyword tells compiler object will be statically initialized with a constant value, solving the static initialization order issue

### concept
concept make sure data used within a template fufill a specified set of criteria. concept can also be applied to return type, consider it c++ void * type

stl provides basic set of concepts

### std::format
providing python string formating functionality to c++

### source location
replacing __FILE__ and __LINE__ with something more useful

### modules
replacing #include.

### synchronization library

### spaceship operator <=>

### using enum
to get rid of some namespace separation noise

### partial deprecation of volatile
