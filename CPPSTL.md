### Beautiful C++ STL
#### Counting
count STL function returns a number of object match the target. e.g.
```
std::array<int> a = {1,2,3,4,5};
size_t r = std::count(std::begin(a), std::end(a), 5); //count how many 5 are in a
```

STL contains generic function `begin` and `end` to return the begin and end of iterator. STL algorithms only act on the iterator of the container, not directly on the container itself.

count_if returns number where a condition has been true. e.g.
```
std::array<int> a = {1,2,3,4,5};
size_t r = std::count_if(std::begin(a), std::end(a), [](int v){ return v % 2 == 0; }); //count odd numbers in array
```

but most of the the time we are only interested in the logical conditions from a container if it contains any element, contains all the same element, or contains different element. so we can replace count and count_if with `std::all_of`, `std::none_of`, and `std::any_of`

#### Finding
multiple functions for find
```
find
find_if
find_if_not    # reverse the logic of find_if
find_first_of
search         # difference between search and find is find targets 1 element of the collection, search target a sequence
find_end       # this is really search but from the end, not find from the end, reverse of search function
search_n       # search for n occurrence of a sequence in collection
adjacent_find  # search the collection for 2 consecutive equal elements, can be any element
```

all functions return an iterator

#### Sorting
```
sort
stable_sort  # slower algorithm but will make sure equal elements are also going to be in the same order after stable_sort
is_sorted    # returns if the collection is already sorted
partial_sort # only have the unstable version, can be an optimization when you don't have to sort the full collection. rearrange elements
             # in such way as smallest elements before the middle operator are sorted, other elements between middle and end are not
is_sorted_until # examine the range [first, last) and find the largest range begining at first in which the elements are sorted in ascending order
partial_sort_copy # copies the smallest elements from [first, last) to [result_first, result_last), sorting the elements copied
nth_element  # rearranges the container such that the nth element as if the container is sorted will be at the nth position, leaving the rest in
             # unspecific order, except that none of the elements preceeding the n-th element are greater and none of the elements after nth are smaller

```

STL does not provide way to automatically choose optimal algorithm on container based on intrinsic properties such as whether the container is sorted. e.g. sorted list vs. unsorted list. Instead, user must choose the appropriate algorithm based on prior knowledge whether the container is sorted or not.

|algorithm | sorted | unsorted |
|----------|--------|----------|
| obtain the maximum element | use begin() or end()                                                           | use max_element() function which is O(n) |
| obtain the minimum element | use begin() or end()                                                           | use min_element() function which is O(n) |
| finding an element         | use upper_bound() or lower_bound(), which can only be done on sorted container | use find, which is O(n) |

sometimes, the application may not need to sort an entire container, it may only obtain the first 5 largest element, sorting the entire container would not be optimal. The stl is designed in such way this can be done accordingly.



#### Shuffle
```
shuffle # create a random reordering of the container
```

shuffle requires `<random>` header, need to use generator such as mt19937 where 19937 is the prime used for the mersene twister, need to use a random device such as the the default random device.

```
random_device rd;
mt19937 generator(rd());
shuffle(b, e, generator);
```

#### mismatch
```
mismatch # compares the elements in [first, last) with elements begining at first2, returns the first element that does not match
```

problem: what if second container is smaller than first container ? behavior is probably undefined. however, there is interface that requires end2.

mismatch returns a pair, which is a pair of iterators, one from each array where the mismatch happened. elements are compared using ==

#### accumulate
```
accumulate # doing an operation iteratively from begining and end, with a starting element
```

accumulate can also be used to multiple all elements together, not just add, by overriding the operator to do multiplication. there are also other potential uses, such as concatenate strings together.

#### for_each
```
for_each
```

a way to avoid having to write a loop. this is best used when you don't have to apply an operation to the entire collection. when it applies to the entire collection, it would be better to use the for each element looping syntax.

adding a + in front of a lambda expression turns the lambda expression from a lambda literal into a function pointer type

#### copying
```
copy
copy_if
copy_n  #copy from one container to another for n elements, if n is bigger than destination container or input container, this is undefined
copy_backward
```

if it is copying the entire collection, use copy constructor. 

copy within a collection is safe if the begin and end ranges do not overlap. if it does, it is safe to use copy_backward if the begining of the destination iterator is before the end of the input iterator.

copy_backward copies starting from the last element needed to be copied. it is often used only for the copying within the same container, shifting the elements to the right a little. if it is to shift elements to the left, only need to use the copy() function.

copy_backward is awkward as the destination iterator is taken as the end() iterator, and it is accessed in reverse order to begin.

it is undefined behavior if destination does not have enough size to hold onto the entire copied collection

#### move
```
move          # move a segment of a container element to another container, leaving empty shells in the original, use the move constructor
move_backward # move but backward within the container. it is odd that standard included this algorithm but not move_if and move_n
```

#### remove
```
remove    # move elements matching critiera to the end of the container, giving a iterator to the begining of the removed elements
remove_if
```

`container.erase(remove(...))` is a common idiom. it means to erase some elements in container that fits critiera. erase and remove are separate in c++ for optimziation reasons.

#### create and fill collections
```
fill
fill_n
iota       # generate values by adding 1 at a time
generate   # takes a lambda, can generate anything
generate_n
```

these are best to use when construction is only needed for a certain segment of the container, but not all.

#### replacing values
```
replace
replace_if
```

#### transform
```
transform #can take consecutive elements in an container and do transformations too
```

#### eliminate duplication
there are multiple ways to eliminate duplicate within a container
* sort() then unique(), where unique() only look at consecutive values, returns a iterator pointing to list of elements to be removed, container is dirty after
* sort() then unique_copy(), instead of dirty the container, unique_copy copies the unique elements to a destination container

#### reverse and swap
```
reverse      #reverse a container, the container is then modified
iter_swap    #swap the values pointed to by 2 iterators
reverse_copy #do not modify the container
```

#### iterators
header ``<iterator>``

```
back_inserter
front_inserter
```

inserters depends on whether a container has push_back or push_front methods; will fail if container does not provide those methods.

inserters only work with algorithms where it only take the begin of a destination output container, not the end of the destination output container. this means only functions such as `fill_n`, `generate_n`, `reverse_copy` etc.

#### iterator arithmetic
```
begin()
end()
cbegin()
cend()
rbegin()
rend()
crbegin()
crend()
```


