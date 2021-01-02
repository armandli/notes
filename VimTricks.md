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

### move between change list
g; and g,

### move between methods
[m and ]m

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
:Termdebug program
:help terminal-debug

#### Visual Modes
v, V, <Ctrl-V>
for visual character mode, visual line mode, and visual block mode

#### Insert Mode Triggers
i, a, c

#### Command Mode Triggers
:, /

#### vim -u to avoid all config

#### Do diff on all panes
:windo difft

#### think operators, text objects, and motions
operators are c, d, y, ~, gu, gU, !, <, >, =
c for change
d for delete
y for yank into register
~ for swap case
gu for make lowercase
gU for make uppercase
! for filter to external program
< for shifting left
> for shifting right
= for indent

to documents on operators, use :h <operator>

text objects are
aw for a word, and the space that follows
iw for inner word
aW for a WORD
iW for inner WORD
ap for a paragraph
ip for inner paragraph
ab for a bracket
ib for innert bracket
at for a tag block
it for inner tag block

motions are
% for go to first matching parenthesis
[count]+ for down to first non-blank char of line
[count]$ for to end of line
[count]f/F{char} for to next occurrence of {char}
[count]t/T{char} for to before next occurrence of {char}
[count]h/j/k/l for left down up or right
[count]]m for go to begining of next method, work for python
[count]w/W for go a word/WORD to the right
[count]b/B for go a word/WORD to the left
[count]e/E for go to end of word/WORD right

to put it all together
"[count][operator][text object/motions]"

examples:
6+ for go to line 6 times
gUaW for capitalize a word
3ce for change to wrod end by 3 times
4$ for go to end of line by 4x
d]m for delete to start of next method
% for jump to match of next paren or bracket

#### various motions
H for high, M for middle, L for low , move such that the cursor is in the middle
Ctrl-U for go up, Ctrl-D for go down half screen, Ctrl-B for go up, Ctrl-F for go down for a full screen, Ctrl-E and Ctrl-Y for go line up and down
zt, zz, zb for put current curosr position top top, middle buttom

#### Editing
:e[dit] [++opt] [+cmd] {file}
:fin[d][!] [++opt] [+cmd] {file}  for find file to edit
gf for go to file
Ctrl-^ to switch buffer

#### Search
/{patt}[/]<CR> to search for pattern
/<CR> for search for last used pattern
?{patt}[?]<CR> for search back for last used pattern
[count]n for repeat last search [count] times forward
[count]N for repeat last search [count] times backward
* for search forward for word under cursor
\# for the same as * but opposite direction
gd for go to local declaration
:hls! for toggle searh highlight

#### Marks
m{a-zA-Z} set a custom mark whose exact location can be accessed using '{mark} and lin accessed using `{mark}
:marks to show all current marks used
certain marks are special, `. and '. are used for jumping to the last change

#### Tags
Ctrl-] and Ctrl-t to jump within projects
Ctrl-O and Ctrl-I are cycle through :jumps, can jump between files
g; or g, are cycle through :changes , for go to previous change

create index based off current working directory
:!ctags -R --languages=python --exclude="env"

after tags are created, we can do g Ctrl-] to show all places it sees the keyword. without the g it will go to the first one it sees, Ctrl-O to jump back

#### to go file in text
gf

#### creata terminal buffer
:bo 15sp +te
or
:ter

:ter ++curwin makes the current buffer a shell

#### vim buffers
tabs are window containers
windows are for buffer viewports
buffers are for file proxies and the arglist

sample buffer actions are
:bn for go to next buffer
:b {filename} for go to buffer {filename}
:bd for delete current buffer
:buffers to print out all buffers
:bufdo {cmd} for execute {cmd} for all buffers
:n for go to next file based on arglist
:arga {filename} for add {filename} to arglist
:argl {files} for make a local copy via {files}
:args to print out all arguments

the argument list is a stable subset of the buffers list, which can help manage modules

sample windows actions are
<Ctrl-w> s for split window
<Ctrl-w> v for split window vertically
<Ctrl-w> q for close window
<Ctrl-w> w for alternate window
<Ctrl-w> r for rotate window
<Ctrl-w> x for change view port
<Ctrl-w> o to make the current buffer to span the entire window
:windo {cmd} for execute command on all windows
:sf {file} for split window and :find {file}
:vert {cmd} for make any split {cmd} be vertical

tabs sample actions are
gt to go to next tab
gT to go to previous tab
:tabc to close tab
:tae to open tab
:tabo for close all other tabs
:tabf to do create a new tab and find a file to show

#### create a directory tree pane
:20vs .  to show the files in the current directory
in the directory tree, we can hit p to preview

can change the arglist to be all files of certain pattern, e.g. yaml
:args **/*.yaml

:sall to split all buffers or arglist
:vert sall to vertically split

:vert sf <pattern> to find a file and vertical split on it

auto file completion by doing Ctrl-X Ctrl-L to copy from left vertical buffer

:vim /TODO/ % to search for TODO in all files matching the current file type, open them in buffers

:cn for quick fix list

@: to repeat a command

:vim /TODO/ ## to represent everything in my arglist, note ## is a symbol

:cdo s/TODO/DONE/g is for repeated operation on every buffer to replace TODO with DONE

#### clipboard synchronization
OSC 52, terminal (xterm) recognizes certain escape sequence called operating sequence which will send a signal to the parent terminal tty, which will do something
to your terminal, in this case, 52 allows synchronization of terminal clipboard
\e\e]52;c;(b64_data)\x07\e\\'

vimrc function, copy yank buffer to system clipboard, use OSC2 to put things into the system clipboard, work over ssh
```
function! Osc52Yank()
  let buffer=system('base64 -w0', @0) " -w0 to disable 76 char line wrapping, copy to register 0
  let buffer='\ePtmux;\e\e]52;c;'.buffer.'\x07\e\\'
  silent exe "!echo -ne ".shellescape(buffer)." > ".shellescape(g:tty)
endfunction
nnoremap <leader>y :call Osc52Yank()<CR>
```

#### vim registers
:registers to show all register values

#### screen multiplexing with tmux
multiplex project into different window panes, instead of a single vim session, we can have multiple vim sessions within 1 window, in this case the screen window is
different from the vim window

#### sort lines
:sort

we can also sort on patterns within each line. e.g.

:sort /-\w*/ r
:sort /(.*)/ r

#### matching repeated words
`\<\(\w\+\) \1\>`

we can use it in substitution `%s/\<\(\w\+\) \1\>/\1/g`

it is possible to use lookahead syntax in vim `@=` to find words that appear within a given distance of another. example

`\<\(\w\+\)\>\(.\{0,50}\<\1\>\)\@=`
