## Transform any recursive function into a tail call recursion using CPS-transform
we can transform any recursion function into a tail recursion by using the continuation transformation using the following recipe:

1. pass each function an extra parameter cont
2. whenever the function return an expression that doesn't contain function call, send the expression into the continuation instead
3. whenever a function call occurrs in a tail position, call the function with the same continuation cont
4. whenever a function call occurrs in an operand non-tail position, instead perform this call in a new continuation that gives a name to the result and continues
   with the expression

example factorial:
```
def fact(n):
  if n == 0:
    return 1
  else:
    return n * fact(n - 1)
```

transformed:
```
def fact(n, cont):
  if n == 0:
    return cont(1)
  else:
    return fact(n - 1, lambda value: cont(n * value))
```

example fibonacci:
```
def fib(n):
  if n < 2:
    return 1
  else:
    return fib(n - 1) + fib(n - 2)
```

transformed (partial), with a problem:
```
def fib(n, cont):
  if n < 2:
    return cont(1)
  else:
    return fib(n - 1, lambda value: value + fib(n - 2, cont))
```
the problem is the lambda is not a tail call, and we have to recursively transform it further:
```
def fib(n, cont):
  if n < 2:
    return cont(1)
  else:
    return fib(n - 1, lambda value: fib(n - 2, lambda value2: cont(value + value2)))
```

example merge sort:
```
return merge(merge_sort(lst[:mid]), merge_sort(lst[mid:]))
```

transformed into:
```
def merge_sort(lst, cont):
  n = len(lst)
  if n <= 1:
    return cont(lst)
  else:
    mid = int(n / 2)
    return merge_sort(lst[:mid], lambda v1: merge_sort(lst[mid:], lambda v2: cont(merge(v1, v2))))
```

once we transformed recursions into tail recursion using CPS method, we use a trampoline to make sure only one function stack is ever active. trampoline is a loop
that iteratively invoke thunk-returning functions (continuation-passing style). a single trampoline suffices to express all control transfer of a program.

a thunk is a expression without any argument that is wrapped into a function that is not executed until it is used.

trampoline in python:
```
def trampoline(f, *args):
  v = f(*args)
  while callable(v):
    v = v()
  return v
```

we need to turn functions further into a thunk returning function for the trampoline to work. example:
```
def fact(n, cont):
  if n == 0:
    return cont(1)
  else:
    return lambda: fact(n - 1, lambda value: lambda: cont(n * value))
```

so we just trap the tail clals in an argument-less lambda. to invoke the function:
```
trampoline(fact, 6, end_cont)
```
