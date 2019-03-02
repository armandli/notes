###turning one pane into single line pane (useful for csv header)
```
:sp
:0
1 CRTL-W _
CRTW-W
```

### Scroll Binding for multiple Panes
```
:set scrollopt=hor
:set scrollbind
CRTL-W k
:set scrollbind
CRTL-W j
```

### merge multiple lines together (remove line endings)
```
:%s/\r/ /
```

or

```
:%s/\n/ /
```

depending on the line ending of the file, sometimes \r works sometimes \n

### go to the next character in the same line
t<char> forward until next occurence of the character
f<char> forward over the next occurence of character
T<char> backward
F<char> backward

### go to last place of insert
gi go to last place where text insert happened
g;
g,

### change command
c2w delete the next 2 words and enter edit mode, this can be repeatable with . command

### repeat vim macro call
@@.

### repeat the last substitution in the same line
&
g& repeat it to all lines

### repeat command or search
@: repeat command
n/N repeat search

### search command history
q:
q/
