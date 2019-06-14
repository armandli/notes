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

### rerun the last command
":,@:
@: reruns the last command
": is the register storing the last executed command, ":p prints it

### the expression register
"=
can input any vimL expression here and paste it, use ctrl-R etc. e.g. paste the local timestamp by typing "=strftime("%c")<cr>p where <cr> is literally the enter key

### bookmark line
mA sets a mark at the curent cursor to register A, lower case register is per buffer, upper case register is global
'A jumps to bookmark register A from global register
:marks to view all marks

### moving between jumplist
ctrl-I and ctrl-O

### Incremen, decrement
ctrl-A and ctrl-X

### go to begining of visual mode visual
v_o

### visual mode increment by one for each matching line
g ctrl-A/ctrl-X

example:
a 0
b 0
c
d 0

will become:
a 1
b 2
c
d 3

### 3 visual modes:
v is character wise visual mode, V is line wise visual mode, ctrl-V is block wise visual mode

example: d<c-V>2j would convert the motion to blockwise and delete the coloum 

### match nth line below or above the match
/regex/{n}
also have the side effect of making the motion linewise, so we can do d/regex//0 to match regex and delete the line

### runs the ex command only on matching lines on regex
:g/regex/ex
conversely, run ex on lines that does not match the regex
:v/regex/ex

### runs ex on all windows
:windo {ex}
example: :windo $ will scroll all windows to the bottom
there is also :bufdo, :cdo, :tabdo

#### Using GDB to debug in Vim
:packadd termdebug
:TermDebug program
:help terminal-debug
