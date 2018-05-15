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
