# Python Decorator
decorator in python is a wrapper function that add additional functionality without having write additional code. classic python decorator such as adding enter exit
logs on functions, performance testing, transaction processing, permission checking, etc.

example decorator:
```
def debug(func):
  def wrapper(*args, **kwargs):
    print '[DEBUG]: enter {}'.format(func.__name__)
    return func(*args, **kwargs)
  return wrapper

@debug
def say_hello():
  print 'hello!'
```

which is equivalent to (before python 2.4)

```
def debug(func):
  def wrapper(*args, **kwargs):
    print '[DEBUG]: enter {}'.format(func.__name__)
    return func(*args, **kwargs)
  return wrapper

def say_hello():
  print 'hello!'

say_hello = debug(say_hello)
```

decorator can take parameters itself

```
def logging(level):
    def wrapper(func):
        def inner_wrapper(*args, **kwargs):
            print "[{level}]: enter function {func}()".format(
                level=level,
                func=func.__name__)
            return func(*args, **kwargs)
        return inner_wrapper
    return wrapper

@logging(level='INFO')
def say(something):
    print "say {}!".format(something)

# If you don't use the @ syntax
# say = logging(level='INFO')(say)
```

the decorator function is actually an interface constraint that takes a callable object as parameter and returns a callable object. callable objects in python are functions. an object will be callable as long as the object override the __call__() method. you can implement decorators as a class:

```
class logging(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print "[DEBUG]: enter function {func}()".format(
            func=self.func.__name__)
        return self.func(*args, **kwargs)
@logging
def say(something):
    print "say {}!".format(something)
```

and the class can take additional parameters:

```
class logging(object):
    def __init__(self, level='INFO'):
        self.level = level

    def __call__(self, func): # Pass a function
        def wrapper(*args, **kwargs):
            print "[{level}]: enter function {func}()".format(
                level=self.level,
                func=func.__name__)
            func(*args, **kwargs)
        return wrapper  #Return a function

@logging(level='INFO')
def say(something):
    print "say {}!".format(something)
```

## builtin decorators
`@property` which adds a getter, setter, deleter to a function
`@staticmethod` returns a static method object
`@classmethod` returns a class object method

## Problems with Decorator
* wrong function signature and documentation. the object with decorator becomes wrapped in, and so is its documentation and signature. to avoid this, use the
    functool wraps

```
from functools import wraps

def logging(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        """print log before a function."""
        print "[DEBUG] {}: enter {}()".format(datetime.now(), func.__name__)
        return func(*args, **kwargs)
    return wrapper

@logging
def say(something):
    """say something"""
    print "say {}!".format(something)

print say.__name__  # say
print say.__doc__ # say something
```

* cannot decorate a staticmethod, which you do, it fails with exception. this is because @staticmethod does not return a callable object, it returns a staticmethod

```
class Car(object):
    def __init__(self, model):
        self.model = model

    @logging  # Decorate the instance method，OK
    def run(self):
        print "{} is running!".format(self.model)

    @logging  # Decorate the static method，Failed
    @staticmethod
    def check_model_for(obj):
        if isinstance(obj, Car):
            print "The model of your car is {}".format(obj.model)
        else:
            print "{} is not a car!".format(obj)
```

to fix this, you need to reverse the order of the decorator, so logging does not wrap around staticmethod

```
class Car(object):
    def __init__(self, model):
        self.model = model

    @staticmethod
    @logging  # Decorate before @staticmethod，OK
    def check_model_for(obj):
        pass
```

## library helpers for decorators
can use decorate package to implement decorators

```
from decorator import decorate

def wrapper(func, *args, **kwargs):
    """print log before a function."""
    print "[DEBUG] {}: enter {}()".format(datetime.now(), func.__name__)
    return func(*args, **kwargs)

def logging(func):
    return decorate(func, wrapper)  # Decorate func with wrapper.
```

wrapt is a functional package that implement variety of decorators

```
import wrapt

# without argument in decorator
@wrapt.decorator
def logging(wrapped, instance, args, kwargs):  # instance is must
    print "[DEBUG]: enter {}()".format(wrapped.__name__)
    return wrapped(*args, **kwargs)

@logging
def say(something): pass
```

note the parameter list is fixed: (wrapped, instance, args, kwargs) and must be ordered this way. to use additional parameters for the wrapt decorator:


```
def logging(level):
    @wrapt.decorator
    def wrapper(wrapped, instance, args, kwargs):
        print "[{}]: enter {}()".format(level, wrapped.__name__)
        return wrapped(*args, **kwargs)
    return wrapper

@logging(level="INFO")
def do(work): pass
```

