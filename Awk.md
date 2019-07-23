## Awk Notes
option `-F` sets the delimiter within the file, this can be a text line
awk takes a '' string as command, within the command, /pattern/ help remove the pattern from the column. {print $2} prints the entire column after pattern removal
example:
```
cat file | awk -F "Error: " '/Error/ {print $2}'
```
