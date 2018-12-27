### Bash Completion
bash completion is done using the `complete` command which completion suggestions can be displayed for given executable. completion can simple to sophisticated.

example completion for command `dothis <#>` will execute the command in history #.

dothis command code:
```
if [ -z "$1" ]; then
  echo "No command number passed"
  exit 2
fi

exists=$(fc -l -1000 | grep ^$1 -- 2>/dev/null)

if [ -n "$exists" ]; then
  fc -s -- "$1"
else
  echo "Command with number $1 was not found in recent history"
  exit 2
fi
```

for copletion, create file `dothis-completion.bash` as the completion script. sourcing this script will allow completion to take effect

#### static completion
suppose dothis support static command such as `now`, `tomorrow` and `never`. we can use the `complete` command to regiser for bash completion:

```
#/usr/bin/env bash
complete -W "now tomorrow never" dothis
```

this will be generated when we access completion after typing `dothis` or `dothis n`

there are many other static completion options, such as `-A` to display directory names after the command:

```
complete -A directory dothis
```

here is the complete [list](`https://www.gnu.org/software/bash/manual/html_node/Programmable-Completion-Builtins.html#Programmable-Completion-Builtins`)

#### dynamic completion
instead if want the behavior for dothis to be:
* if user did not enter any number, display the last 50 commands in history
* if user typed some number sequence that matches more than 1 commands from history, display those command and their number
* if user typed some number that match exactly one command in history, autocomplete the typing

start with completion script:
```
#/usr/bin/env bash
_dothis_completions()
{
  COMPREPLY+=("now")
  COMPREPLY+=("tomorrow")
  COMPREPLY+=("never")
}

complete -F _dothis_completions dothis
```

where -F flag indicate the function `_dothis_completions` will act as completion function for dothis. COMPREPLY is a array variable storing the completions. the completion mechanism uses this variable to display its content as completion. COMPREPLY variable disables automatic filtering, so all logic are now controlled by the user inside the function.

to allow for filtering, use command `compgen` e.g. `compgen -W "now tomorrow never" n` will filter for only now and never

bash completion facilities provide bash variables for supporting completion:
* `COMP_WORDS` is an array of all words typed after the name of the program the `compspec` belongs to
* `COMP_COWRD` is an index of `COMP_WORDS` array pointing to the word the current cursor is at, or the index of the word where completion is triggered
* `COMP_LINE` is the current command line

changing the bash completion script for dothis command to:
```
#/usr/bin/env bash
_dothis_completions()
{
  COMPREPLY=($(compgen -W "now tomorrow never" "${COMP_WORDS[1]}"))
}

complete -F _dothis_completions dothis
```

now the function `_dothis_completions` does exactly the behavior of option -W

this completion script will achieve objective (1) where we display the last 50 commands
```
#/usr/bin/env bash
_dothis_completions()
{
  COMPREPLY=($(compgen -W "$(fc -l -50 | sed 's/\t//')" -- "${COMP_WORDS[1]}"))
}

complete -F _dothis_completions dothis
```

but there is a problem: if we have more than 1 option specified, the display also works. we can disable completion using the following:
```
#/usr/bin/env bash
_dothis_completions()
{
  if [ "${#COMP_WORDS[@]}" != "2" ]; then
    return
  fi

  COMPREPLY=($(compgen -W "$(fc -l -50 | sed 's/\t//')" -- "${COMP_WORDS[1]}"))
}

complete -F _dothis_completions dothis
```

the final script that can achieve all 3 objectives above:
```
#/usr/bin/env bash
_dothis_completions()
{
  if [ "${#COMP_WORDS[@]}" != "2" ]; then
    return
  fi

  # keep the suggestions in a local variable
  local suggestions=($(compgen -W "$(fc -l -50 | sed 's/\t/ /')" -- "${COMP_WORDS[1]}"))

  if [ "${#suggestions[@]}" == "1" ]; then
    # if there's only one match, we remove the command literal
    # to proceed with the automatic completion of the number
    local number=$(echo ${suggestions[0]/%\ */})
    COMPREPLY=("$number")
  else
    # more than one suggestions resolved,
    # respond with the suggestions intact
    COMPREPLY=("${suggestions[@]}")
  fi
}

complete -F _dothis_completions dothis
```

you can improve the experience by displaying each suggestion in its own line using printf:
```
#/usr/bin/env bash
_dothis_completions()
{
  if [ "${#COMP_WORDS[@]}" != "2" ]; then
    return
  fi

  local IFS=$'\n'
  local suggestions=($(compgen -W "$(fc -l -50 | sed 's/\t//')" -- "${COMP_WORDS[1]}"))

  if [ "${#suggestions[@]}" == "1" ]; then
    local number="${suggestions[0]/%\ */}"
    COMPREPLY=("$number")
  else
    for i in "${!suggestions[@]}"; do
      suggestions[$i]="$(printf '%*s' "-$COLUMNS"  "${suggestions[$i]}")"
    done

    COMPREPLY=("${suggestions[@]}")
  fi
}

complete -F _dothis_completions dothis
```
