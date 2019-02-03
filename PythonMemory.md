# CPython Memory Usage and Optimizations

tool: [gdb-heap](github.com/rogerhu/gdb-heap) for analyzing python memory usage

adding C extensions to python could lead to memory leak due to extension implementation

In Python2.6, we can get object memory usage in bytes, excluding other objects referenced using:
```
sys.getsizeof(obj)
```

### General Memory Usage:
|Type                           | 32-bit              | 64-bit              |
|-------------------------------|---------------------|---------------------|
|int python2                    | 12                  | 24                  |
|long python2 or int python3    | 14 + (2 * digits)   | 30 + (2 * digits)   |
|str python2 or bytes python3   | 24 + len            | 40 + len            |
|unicode python2 or str python3 | 28 + (2 or 4 * len) | 52 + (2 or 4 * len) |
|list                           | 32 + (4 * len)      | 72 + (8 * len)      |
|tuple                          | 24 + 4 * len        | 64 + 8 * len        |
|float                          | 16                  | 24                  |

general trend is 64-bit consumes twice as much memory compared to 32-bit. Python could use a lot of memory at different levels: Python Level, Library/Extension Level

dictionary consume more memory:

|size of dictionary | 32-bit                                | 64-bit                                |
|-------------------|---------------------------------------|---------------------------------------|
|base               | 136 + 12 * external PyDictEntry Table | 280 + 24 * external PyDictEntry Table |
|0-5 entries        | 136                                   | 280                                   |
|6-21               | 520                                   | 1048                                  |
|22-85              | 1672                                  | 3352                                  |

dictionary size quadruples in size because it needs to make sure portion of its map is free

slots optimization: adding `__slots__` to python class could reduce memory usage of the class by 4/5 e.g.

```
class NewStyle(object):
  __slots__ = ('line1', 'line2', 'apartment', 'city', 'state', 'zipcode')
  def __init__(self, line1, line2, apartment, city, state, zipcode):
    self.line1 = line1
    self.line2 = line2
    self.apartment = apartment
    self.city = city
    self.state = state
    self.zipcode = zipcode
```

it limits what the attribute in the python object would ever have, helping CPython reduce memory

reduce python memory usage at the Python Level, use tool [Meliae](https://launchpad.net/meliae), it's a python extension for taking snapshot of object reference graph, as seen by GC, dumps object reference graph in JSON, tool to analyze result

to analyze memory usage of Python built-in types, `PyObject_Malloc` uses malloc, which carves up 256KB of RAM which is then carved further into 4KB. lots of optimization for small object allocations. in GNU libc, it implements malloc using boundary tags implementation, multiple boundary region kept track of using sbrk and anonymous mmap. which is a doubly-linked list. looking at /proc/ppid/maps will show you memory regions labeled "heap" but also unlabeled regions that are also heap, specifically anonymous map.

to analyze memory usage, use valgrind would simulate the program, which changes the program behavior. gdb-heap is a gdb extension, gdb7 can use python as script that introspect data in program being attached to. 

### Avoid String Copying in Python
example: reading a file, and referring to a segment of the file when reading would cause the file to be copy out the referred segment. to avoid the copy out when reading, use objects that implements the buffer protocol. objects implement this protocol have a memoryview() function that you can use when constructing the object. this will give a memoryview object on the original object as output. e.g.

```
s = b'abcdefg'
view = memoryview(s)
slice = view[1:5] #does not cause a copy on read
```

example usage: sending out data through socket. when sending out data through socket, the data being sent out are copied before sending out. use memoryview to avoid
the copy.

```
import socket
s = socket.socket(…)
s.connect(…)
# Build a bytes object with more than 100 millions times the letter `a`
data = b"a" * (1024 * 100000)
mv = memoryview(data)
while mv:
    sent = s.send(mv)
    # Build a new memoryview object pointing to the data which remains to be sent
    mv = mv[sent:]
```

both objects in the array module and functions in the struct module handle buffer protocol

### use preallocated buffer for read
when reading a file, you can preallocate a buffer to read data into, controlling the maximum amount of memory usage. example

```
ba = bytearray(8)
with open('/dev/urandom', 'rb') as source:
  source.readinto(ba)
```

we can use memoryview here to use part of the buffer to be read into as well.

### memory_profiler module
use memory_profiler module to monitor the memory usage of each line of code in python


