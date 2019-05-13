### Tips for writing cache friendly code

#### pack data structure
the number of bits of information in a data structure needed is different from the total bytes consumed due to alignment rules. compact data structures are more
cache friendly.

e.g. a pointer is only 62 bits of information (due to memory address alignment rule) on 64 architecture. a enum has size int by default but its bits of information
could be much smaller than max value of an int

#### use std::variant for inheritance hierarchy
use std::variant to keep track of all possible subclass used in a inheritance hierarchy for cache friendly data structure and avoid pointer. e.g.

```
struct Button {
  string label;
  variant<Circle, Square> shape; //Circle and Square are subtypes of Shape
};
```

#### Use pointer to data in a data structure
if a data structure is used for search, and only part of the data structure represent they key, push the remainder as a pointer to that data to make it cache
friendly.

#### Cache Friendly Container
don't use std::list and std::set, use vector, or implement other data structures on top of the vector

vector insert can be done using push back new element to the end, and then rotate the segment between inserted index and the end inclusively

vector erase can be done by replacing the element to be erased with the last element in the vector. if they are equal, move will trigger a deconstruction (work as
expected)

for frequent insert and erase but order is not important situations, use vector instead of list, use vector with a bag interface, where we replace insert and erase
as O(1) operation.

for frequent lookup withere order is not important, use a vector instead of a set or unordered set. use vector with a set interface, where insert is a push back, and find is just a linear search, or a sorted set interface, or design a cache friendly hash set.

for frequent lookup elements by some external property where order is not important, use a set with vector of values instead of map or unordered map. example:
```
template <typename Key, typename T>
class sorted_map {
  sorted_set<Key> keys; //also implemented as a vector
  std::vector<T> values;

  T& operator[](const Key& key){
    auto iter = keys.find(key);
    return values[index_of(key)];
  }
};
```

customization point of a vector:
1. how memory is going to be allocated
2. new capacity after a push back

to customize allocation, use allocator. or polomorphic memory resource.
cannot customize growth, maybe add a new policy, a growth policy, but allocator and growth are tightly coupled.

small vector optimization. first few elements can avoid a heap allocation. can also use `array<T>` as the underlying implementation with fixed small vector optimization. use `array<T>` to implement `vector<T>`, `bag<T>`, `flat_set<T>`, `flat_map<K, V>`, `HashMap<K, V>`

use BlockStorage for Array

#### Code is memory too
best to execute code that has no branch, no jump, no function call, especially virtual function call. but this is not possible in all situations. instead, avoid them in algorithm implementation.

make sure hot code are close to one another in a loop so there is fewer instruction cache line load.
rearrange branches so that code that least likely to execute are bunched together in its own instruction cache.
avoid function pointers and virtual function calls because if they are not optimized, it's a lot of code jump.
use attributes such as hot/cold in GCC/clang and likely/unlikely attributes in C++20 to tell compiler which branch is more frequently executed

code cache line alignment can be improved by the location of the function in the program; sometimes using NOP to help align the code or align the data to be cache
aligned will speed up the code.

#### cache friendly program structure
problems: cache unfriendly indirection such as std::unique_ptr; wasted cache line space for alignemnt; function pointer in hot loop; invisible intities using up
code cache.
solution: data oriented design

group data based on how it is going to be used, not based on real world entity concept.
implement the common task, not the special case.
eliminate boolean and object state by making them implicit.

instead of array of objects, do object arrays where each single attribute of all data object is in its own array. this way there is no wasted space for padding, and we access attributes by how it is used, not how it represents real world entity. i.e. struct of array vs array of struct. it's also useful because we can use SIMD instructions and other reasons

better way to represent struct of array is to have a type like `tuple_bag<T1, T2, ...>` which is a wrapper of multiple arrays for each T, all arrays have the exact same size

