## simple implementation to replace std::move and std::forward

for std::move
```
Type t = static_cast<Type&&>(v);
```

for std::foward
```
template <typename Fn, typename ... Args>
void call(Fn fn, Args ... args){
  fn(static_cast<Args&&>(args) ...);
}
```
move is 100% correct, but I'm not sure about foward replacement