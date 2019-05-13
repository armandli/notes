## Anti-Patterns in Python
1. should use `with` instead of open() and close()
2. do not use type in variable names
3. a function should not return more than 1 return type
4. should use get() on dictionary to avoid key not found conditional
5. use items() on dictionary instead of iterate through keys and then search the said dictionary with the key for its item
6. do not create classes to contain data only, use `namedtuple` from `collections` instead, or attribute classes in python3.7
7. do not define getter and setters for a class, use `@property` and `@member.setter` instead
