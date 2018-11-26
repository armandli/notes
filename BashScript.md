### Safer Bash Scripting
`set -euxo pipefail`
`set -e` exit the shell script when a command fails, add `|| true` for the exception where failure is tolerated
`set -o pipefail` set exit code of a pipeline to the right most command to exit with a non-zero status, or 0 if all commands succeeded
`set -u` treat unset variables as an error and exit immediately; handles special situation `${a:-b}` where a is undefined
`set -x` debug mode on, print every executed command
`set -E` will cause ERR trap to not fire in certain scenarios
