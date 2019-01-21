# is_transparent trait in functors for set comparison without its key type
in C++ set, whenever you would want to search for the existence of a key, you must create a temporary key object. This is can be a tedius process if the key object is complex, while only a small subset of members of the key objects are really used for comparison, while other members still need to be constructed just so we can use the find member function.

This now can be avoided in C++14 by the is_transparent trait inside a functor object used for object comparison. e.g.

```
struct CompareID {
  using is_transparent = void; //example is void, but can also be int or struct or anything.

  bool operator()(const ID& a, const ID& b) const { .. }
  bool operator()(int i, const ID& id) const { .. }
  bool operator()(const ID& id, int i) const { .. }
};
```

note this object enables search by using a integer value for object of type ID. we assume ID is made up of more than just a integer.
it also takes advantage of the double functor trick, which enables search using algorithm functions std::find, std::set_difference on ID sets with just a integer, or between a set of ID and a set of integer.

#### double functor trick
```
struct CompareID {
  bool operator()(const S& s, int i){ .. }
  bool operator()(int i, const S& s){ .. }
};

void foo(const std::set<int>& integers, const std::set<S> structs){
  std::vector<int> diff;
  std::set_difference(integers.begin(), integers.end(), structs.begin(), structs.end(), std::back_inserter(diff), CompareID());
}
```

without the transparent, you will not be able to use int type in the find method in a set of ID, now you can. as well as other functions such as lower_bound, upper_bound, equal_range etc
