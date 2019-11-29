#### copy paste from tmux
* requires turning on vi mode in tmux
* use `<Prefix> + [` to enter view mode in tmux session, and use vi movement commands to move up/down tmux command history
* use `<space>` key to enter visual mode during view mode to select form blocks of lines in the tmux terminal to copy text, use `<enter>` to select
* use `<Prefix> + ]` to paste the selected text on tmux terminal

#### tmux logging plugin
* allows saving of tmux session pane into a log file using <Prefix> + <alt> + <shift> + P

#### useful shortcuts
* should add hot key such as z to zone in and out of a pane in the same window
* should add pane size adjustment hotkeys to adjust pane size left-right and up-down, such as `<prefix> + <arrow>`
* should add hot key to swich pane, such as `<prefix> + {`
* should add hot key to change pane layout, such as `<prefix> + <space>`

#### pull the last command arguments
* use `<alt> + .` to pull the last command's arguments out of the last command and paste to current command, can be used to cycle through

#### pull out command manual
use hot key`<prefix> + ?` to pull out help page in tmux

#### create timer
use hotkey `<prefix> + t` to create a timer in the current pane
