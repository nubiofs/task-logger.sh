task-logger.sh [![Build Status](https://travis-ci.org/posva/task-logger.sh.svg?branch=master)](https://travis-ci.org/posva/task-logger.sh)
===

![img](https://cloud.githubusercontent.com/assets/664177/7904349/fa16e226-07f7-11e5-91d5-7255b2c35930.gif)

Shell library to log tasks with nice output. Supports zsh and bash.

|      Property| Value                                                                       |
|-------------:|:----------------------------------------------------------------------------|
|      Version | 1.3.4                                                                       |
|       Author | Eduardo San Martin Morote                                                   |
|      License | The MIT License                                                             |
| Requirements | `perl` With Time::HiRes (comes by default in almost every Linux and in osx).|

# Example

```sh
#! /bin/bash

# Load the lib
source task-logger.sh

# Print some information message
info "Logs are available at $LOG_DIR"

# Print a message and do the corresponding message
# If the task fails, treat it as an error
working -n "Taking a nap"
log_cmd nap sleep 3 || ko

# Print a summary about what have been done
finish
```

Don't forget to call `finish` at the end (It is not mandatory, but summaries are
cool)!

# Motivation

When writing shell scripts you usually want to perform tasks with minimal output
while logging everything somewhere else in order to check it later.  Critical
tasks should stop the script when they fail. At the end you should have a
summary about errors, warnings and successes.

Everything with colors and unicode to make your shell look
:sparkles:fabulous:sparkles:.

# Features

* :lollipop:Colors
* Timers: every task displays the elapsed time
* Customizable:
  * Change any color
  * Create your own messages functions with 2 lines of code
  * Create your own progress loop (see [Working loop](#working-loop))
* Critical tasks
* Summary
* Can be killed
* Smart cleanup

# Usage

## Initialisation
As every shell lib you start by sourcing it: `source task-logger.sh`

## Messages
The lib has some printing functions defined. Every one of them follows this
structure:

```sh
warning() {
  parse_opt "$@"
  echo ${opts[@]} "${WARNING_COLOR}${args[@]}${RESET_COLOR}"
}
```

You can define your own functions adding more text and changing colors.
You can use `printf` or any other printing functions instead of `echo`.
See [Working loop](#working-loop) for an example.

The available functions are:

* `important`: prints a message with a blue background so it is quite visible
* `warning`: prints a message in yellow
* `good`: prints a message in green
* `bad`: prints a message in red
* `info`: prints a message in gray
* `error`: prints a message in white with red background. best fit for critical
    errors
* `ok`, `ko` and `warn` increments the numbers of successes, errors and
    warnings, respectively.  They also print fancy unicode characters
* `working`: prints the current time with format `[HH:MM:SS]` in `info` color
    and the prints a message in blue. This function is usually called with the
    `-n` option (no end line) and just before calling `log_cmd`.

## Logs
Launch tasks with `log_cmd`:

```sh
# log_cmd [-c] <task-name> <command> || (ko|warn)

log_cmd -c critical-task sleep 2 || ko
log_cmd error-task not-a-cmd || ko
log_cmd warn-task not-a-cmd || warn

# prints a summary
finish
```

## Cleanup
All of the logs are carefully saved in a temporary directory. This directory is
created when task-logger.sh is sourced and stored at `$LOG_DIR`.

When you call the `finish` method the temporary directory is cleaned up if there
were no errors. You can also pass the `--force-cleanup` option to clean the
`$LOG_DIR` anyways.  You can also manually cleanup by calling the `tmp_cleanup`.

If you need to create a new log dir use the `new_log_dir` method. This will
populate the `$LOG_DIR` variable but **will not** clean up the previous
directory. To do so you must either call the `finish` method (which will
eventually clean if there are no errors) or the `tmp_cleanup` method.

## Customization

A good example about how to customize the lib is in my
[blog](http://posva.net/shell/2015/02/03/using-task-loggersh/) where I use
task-logger.sh to log my dotfiles install script while keeping a nice output.

## Working loop

When I talk about working loop I refer to the loop called once a task is
launched. In the first version of `task-logger.sh` I was using `dot_working`,
which can still be used by setting the variable `WORKING` to `dot_working` and
`WORKING_END` to `true`. By default the lib use `turning_circle` and
`turning_circle_end`.  Creating your own functions is easy:

* One have an infinite loop and should call itself (`while true; do
...; done`)
* The other one is called when the loop is stopped. It can be used
to move the cursor backwards like I do in `turning_circle_end` using `printf
"\033[3D"`.

If you implement some cool working loop, please create a pull request so
everyone can enjoy it :smile:.

Here is an example of another working loop:

```sh
spinner() {
  local p n symbols
  p=1
  n=6
  symbols=()
  symbols[1]=" ◜ "
  symbols[2]=" ◠ "
  symbols[3]=" ◝ "
  symbols[4]=" ◞ "
  symbols[5]=" ◡ "
  symbols[6]=" ◟ "

  #trap 'printf "\033[5D "; return' SIGINT
  #trap 'printf "\033[3D "; return' SIGHUP SIGTERM

  printf "   "
  while true; do
    printf "\033[3D${symbols[$p]}"
    ((p++))
    if [[ "$p" > "$n" ]]; then
      p=1
    fi
    sleep 0.1
  done
}

spinner_end() {
  printf "\033[3D "
}

WORKING=spinner
WORKING_END=spinner_end

```

## Releases

You can find release history in CHANGELOG.md.

When updating the lib a new version must be released in a separated commit.
This commit must only contain the following additions:
* HISTORY.md: new release entry
* README.md: update version
* task-logger.sh: update version and date

## TODO

* improve `cygwin` compatibility. It does work at the moment except for the
    missing unicode characters. Ctrl+C is printing as it should
* Use some kind of template for working loops and share some variables with it
    from the `log_cmd` method


## Support on Beerpay
Hey dude! Help me out for a couple of :beers:!

[![Beerpay](https://beerpay.io/posva/task-logger.sh/badge.svg?style=beer-square)](https://beerpay.io/posva/task-logger.sh)  [![Beerpay](https://beerpay.io/posva/task-logger.sh/make-wish.svg?style=flat-square)](https://beerpay.io/posva/task-logger.sh?focus=wish)
