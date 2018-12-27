## Makefile Best Practice
### variable setting
* lazy set: `VARIABLE = value` values within are recursivley expand when variable is used, not when it's declared
* immediate set: `VARIABLE := value` values within are expanded during declaration, avoids recursive variable expansion loop
* set if absent: `VARIABLE ?= value` set VARIABLE only if it doesn't already have value, this can be overriden by environment variables

### Compiler variable
* `CC` is the default compiler variable
* `LD` is the default linker variable
* `CFLAGS` is the default C compile options variable
* `CXXFLAGS` is the default C++ compile option variable
* `CPPFLAGS` is the default preprocessor flags for C/C++ and Fortran compilers
* `ASFLAGS` assembly flags

### use pkg-config to find libraries for linking
pkg-config supports Make, CMake, and meson, it is best to use pkg-config to find libraries in the machine for linking

example:
```
PKG_CONFIG ?= pkg-config
CFLAGS := $(shell ${PKG_CONFIG} --cflags openssl)
LIBS := $(shell ${PKG_CONFIG} --libs openssl)
```

### build and install variables
`PREFIX ?= /usr` sets the PREFIX location to install the binary
`BINDIR ?= ${PREFIX}/bin` binary installation location
`DATADIR ?= ${PREFIX}/share` for read-only architecture dependent data files for the program
`MANDIR ?= ${DATADIR}/man` for man pages
`DESTDIR` is the base installation directory
