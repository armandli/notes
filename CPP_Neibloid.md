### Niebloid
named after Eric Niebler. Niebloid is a global function object that disables ADP on its arguments. This is useful in situations where you want to implement a new function with the same name as some other libray, such as STL, and don't want its overload to be fixed with the STL function.

example of how to implement a Niebloid:

```
namespace mystd
{
    class B{};
    class A{};
    template<typename T>
    void swap(T &a, T &b)
    {
        std::cout << "mystd::swap\n";
    }
}

namespace sx
{
    namespace impl {
       //our functor, the niebloid
        struct __swap {
            template<typename R, typename = std::enable_if_t< std::is_same<R, mystd::A>::value >  >
            void operator()(R &a, R &b) const
            {
                std::cout << "in sx::swap()\n";
                // swap(a, b);
            }
        };
    }
    inline constexpr impl::__swap swap{};
}

int main()
{
    mystd::B a, b;
    swap(a, b); // calls mystd::swap()

    using namespace sx;
    mystd::A c, d;
    swap(c, d); //No ADL!, calls sx::swap!

    return 0;
}
```

#### Reference
[StackOverflow](https://stackoverflow.com/questions/62928396/what-is-a-niebloid)