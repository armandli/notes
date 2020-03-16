### Creating a vector of fixed size with unique pointer to sub type objects

```
template <typename T, typename... Args>
auto init_from_moveable(Args&&... args){
  std::vector<std::unique_ptr<T>> vec;
  vec.reserve(sizeof...(Args));
  (vec.emplace_back(std::forward<Args>(args)), ...);
  return vec;
}

struct MyClassVecFunc {
  const std::vector<std::unique_ptr<Base>> m_vec = init_from_moveable<Base>(
    std::make_unique<A>(),
    std::make_unique<B>(),
    std::make_unique<C>()
  );
};

```


