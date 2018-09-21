### How Python Import Works
* `import` statement search through list of path in `sys.path`
* `sys.path` always include path to script invoked in command line and is agnostic to working directory on command line
* importing package is conceptually the same as importing the package's `__init__.py` file

#### Definition
* **module** is any `*.py` file. its name is the file name
* **built-in module** is a "module" (written in C) that is compiled into python interpreter, and therefore does not have `*.py` file.
* **package** is any directory containing a `__init__.py` file. the directory name is the name of the package. in python 3.3, package does not need the `__init__.py` file
* **object** in python is almost anything: functions, clases, variables

#### what does import do?
when a module is imported, python runs all code in the module file. when a package is imported, python runs the content of `__init__.py` file if it needs to exist (python3.3). all names of objects in the module or `__init__.py` are made available to the importer.

#### how does import search for the correct module or package?
when module named `spam` is imported, the interpreter first search for built-in module with the name, if not found then search for a file named spam.py in a list of directories given by variable `sys.path`. `sys.path` is initialized with the following locations:
* the directory containing the input script, or current directory if no file is specified (empty string in `path[0]` when python is running as interpreter mode)
* `PYTHONPATH`, a list of directory names, with the same syntax as the shell variable `PATH`
* installation dependent defaults

if the script name directly refers to a python file, the file is executed as the **main** module

*this means when running as a python script, it doesn't care about the current working directory, only about the path to the script*

`sys.path` is shared across all imported modules.

after initialization, python program can modify `sys.path`. The directory containing the script being run is placed at the begining of the search path, ahead of the standard library path. this means scripts in the directory will be loaded instead of modules of the same name in the library directory.

built-in modules can be found in `sys.builtin_module_names` and are installation dependent. some built-in modules are `sys`, `math`, `itertools`, and `time`

because the rest of the modules in python standard library come after the directory of the current script, it is possible to override python standard library, but not the built-in modules, but built-in modules are instsallation specific, so not all python behave the same when it comes to overriding standard library.

python import is case-sensitive.

#### function of __init__.py
1. convert a folder or script into a importable package of modules
2. run package initialization code. all objects and functions defined in `__init__.py` are considered part of the package namespace

first time that a package or one of its module is imported, python will execute `__init__.py` file in the root folder of the package if it exists. however, if a directory is not considered a package, such as when running a python file without its full path, then the directory containing the file is not considered a package.

example directory:
```
test/packA/a1.py
test/packA/__init__.py
```
when running `test/packA/a1.py`, `test/packA/__init__.py` will be called, but when `python a1.py` is used `test/packA/__init__.py` will not be called because when python runs a script, its containing folder is not considered a package.

importing a package is equivalent as importing the `__init__.py` file as module in the directory.

#### Using objects from imported module or package
4 syntaxes
1. `import <package>`
2. `import <module>`
3. `from <package> import <subpackage or module or object>`
4. `from <module> import <object>`

let X be name that comes after `import`
* if X is name of module or package, then use of object Y need to be called using X.Y
* if X is variable name, it can be used directly
* if X is function name, function can be called using X()

optionally `as Z` can be added to import statement such as `import X as Z`, this renames X into Z, so X is no longer valid, but Z is.

example: `import packA.subA.sa1` we have to use function `foo()` in sa1 as `packA.subA.sa1.foo()`
example: `from packA.subA import sa1` or `import packA.subA.sa1 as sa1` we have to use function `foo()` in sa1 as `sa1.foo()`

Use dir() to examine content of imported module

#### Absolute vs Relative Import
aboslute use full path to the desired module to import; relative import use relative path (starting from the path of current module) to the desired module to import.

there are 2 types of relative imports
* a explicit relative import follows a format `from .<module/package> import X` where module/package is prefixed by a dot. a single dot means the current directly, a double dot means from a folder up
* implicit relative import as if the current directory is part of sys.path. implicit relative import is only supported by python2, not supported in python3

in general, absolute path is preferred over relative path, and any script that uses explicit relative import cannot be run directly because they are based on the name of the current module, since the name of the main module is always `main`, modules intended to use as main module of a python application must always use aboslute path for import.

#### Misc
* `from <module> import *` does not import names in module starting with `_`
* `if __name__ == '__main__'` is used to check if a script is imported to run directly
* using `__all__` variable in `__init__.py` for specifying what gets imported by `from <module> import *`
* install a project as packge in developer mode using `pip install -e <project>` to add prooject root directory into `sys.path`
