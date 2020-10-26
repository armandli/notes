### C++20 Algorithm with ranges
C++20 reimplemented algorithms with ranges and concepts. the STL algorithm interface has changed. example:

```
template <ranges::input_range R, std::weakly_incrementable O>
requires std::indirectly_copyable<ranges::iterator_t<R>, O>
constexpr ranges::copy_result<ranges::borrowed_iterator_t<R>, O>
copy(R&& r, O result);

template<ranges::input_range R, class Proj = std::identity,
          std::indirect_unary_predicate<std::projected<ranges::iterator_t<R>, Proj>> Pred >
constexpr ranges::borrowed_iterator_t<R> find_if( R&& r, Pred pred = {}, Proj proj = {} );

template< ranges::input_range R, std::weakly_incrementable O,
          class Proj = std::identity,
          std::indirect_unary_predicate<std::projected<ranges::iterator_t<R>, Proj>> Pred >
requires std::indirectly_copyable<ranges::iterator_t<R>, O>
constexpr ranges::copy_if_result<ranges::borrowed_iterator_t<R>, O>
copy_if( R&& r, O result, Pred pred, Proj proj = {} );
```

the interface has 3 major components: the container, a predicate that selects elements from the container using some condition, a projection that transforms each element before being feed into the predicate.

it uses C++20 concept to make sure the result iterator R is able to be copied over to output container O.

all range algorithms are implemented as neibloids, or global function objects with argument dependent name lookup disabled.

with the new algorithm interface, we can compose multiple algorithms together. example:

```
std::vector ints { 1, 2, 3, 4, 5, 6, 7 };
std::ranges::copy_if(ints | std::ranges::views::take(4), std::ostream_iterator<int>(std::cout, ", "),
                     [](int x) { return (x % 2) == 0; });
```

example of how to use range algorithms:

```
#include <algorithm>
#include <iostream>
#include <iterator>
#include <ranges>
#include <vector>

struct Package {
    double weight;
    double price;
};

int main(){
    std::vector<Package> packages {
        {100.0, 10.0},
        {104.0, 7.5},
        {95.0, 17.5},
        {91.0, 15.0},
        {100.1, 12.5 },
    };
    auto print = [](Package& p) { std::cout << p.weight << ": " << p.price << '\n'; };
    std::ranges::sort(packages, {}, &Package::weight);
    std::cout << "by weight: \n";
    std::ranges::for_each(packages, print);
    std::ranges::sort(packages, {}, &Package::price);
    std::cout << "by price: \n";
    std::ranges::for_each(packages, print);
}
```
#### Reference
[Bartek's coding blog ](https://www.bfilipek.com/2020/10/complex-ranges-algorithms.html)