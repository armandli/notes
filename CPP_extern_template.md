## C++ Extern template
extern template is a feature added in C++11. when used in a compilation unit, the compilation unit will assume the instantiation is done externally and does not
generate the instantiation locally.

example header file containing template:
```
template <typename T> void bigfunction(){
  //body
}
```

example cpp file that uses the template function with extern template:
```
// bigfunction with int instantiation is externalized and will not be instantiated in this cpp file
extern template void bigfunction<int>();
```

the use of this feature is to reduce compilation. when the header is included in multiple compilation units, multiple instantiation will be generated, but during
linking, all except for one instance of the specific instantiation is kept and the rest are discarded, wasting compilation time. this is really bad if bigfunction
is large and require a long time to instantiate.

however, the story does not end here. extern template has conflict with function ininling. example: if the only compilation unit containing bigfunction decided to
inline `bigfunction<int>`, this will cause a linking error in other compilation units for missing `bigfunction<int>`. so the flaw of the feature is that extern
template does not promise a instantiation is actually produced. to work around it, we will need explicit instantiation of big function in one of the translation
units, preferably the cpp file of the header.

example explicit instantiation in cpp of header
```
template void bigfunction<int>();
```

this is needed in order to guarantee linking free of errors.
