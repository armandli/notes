### Selection Initialization
C++17 if and switch statement can be used to save additional variable initializations, called **selection initialization**

```
if (auto [a, b] = func(); a < b){
  ~~~
}
```

where a and b can be of different type. a and b are both only valid within the if block
