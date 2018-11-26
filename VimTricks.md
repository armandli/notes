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
