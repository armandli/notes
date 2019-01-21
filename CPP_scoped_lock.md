### C++17 scoped lock
replacing scoped_guard in C++17, there is scoped_lock. it works the same as scoped_guard, except now scoped_lock can lock multiple mutex during its creation.

```
std::mutex mutex1;
std::recursive_mutex mutex2;
std::scoped_lock<std::mutex, std::recurisve_mutex> locks(mutex1, mutex2);
```

in C++17, we can also use constructor argument deducion to avoid having to type the templates too

```
std::mutex mutex1;
std::recursive_mutex mutex2;
std::scoped_lock locks(mutex1, mutex2);
```
