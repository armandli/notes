### Anti-Patterns in Python
1. should use `with` instead of open() and close()
2. do not use type in variable names
3. a function should not return more than 1 return type
4. should use get() on dictionary to avoid key not found conditional
5. use items() on dictionary instead of iterate through keys and then search the said dictionary with the key for its item
6. do not create classes to contain data only, use `namedtuple` from `collections` instead, or attribute classes in python3.7
7. do not define getter and setters for a class, use `@property` and `@member.setter` instead

### Pythonic way of using dictionary
instead of using condition if else for retrieving value for key that may not exist, use get() and setdefault(). get() can set a default if key cannot be found, the
default is returned.
setdefault() help write a key into dictionary only if key does not already exist, removing the need to do if else condition to check if key exist or not

instead of keep on doing if else switches on multiple options, use a dictionary and get from the dictionary
