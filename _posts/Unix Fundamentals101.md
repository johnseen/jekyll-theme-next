---
type: Linux
title: Linux基础
date: 2019-12-01
category: Linux
description: Linux基础
---



# Unix Fundamentals 101

- [Unix Fundamentals 101](#unix-fundamentals-101)
  - [Shell](#shell)
    - [whait is shell](#whait-is-shell)
    - [Introduction to Bash](#introduction-to-bash)
    - [Shell Fundamentals](#shell-fundamentals)
      - [Command-line Editing Modes](#command-line-editing-modes)
        - [Example Edit Commands](#example-edit-commands)
  - [Enviroment variables](#enviroment-variables)
    - [$PATH](#path)
    - [Shell Profiles](#shell-profiles)
    - [Profile Precedence](#profile-precedence)
  - [Special Environment Variables](#special-environment-variables)
    - [Process ID: `$$`](#process-id)
    - [Background Process ID: `$!`](#background-process-id)
    - [Exit Status `$?`](#exit-status)
    - [History](#history)
    - [Re-executing Commands](#re-executing-commands)
    - [Searching History](#searching-history)
    - [Job control](#job-control)
    - [Multiple Jobs](#multiple-jobs)


## Shell 

### whait is shell
shell is the commmand-line interface between the user and system
### Introduction to Bash
Bash is known as the Bourne-again shell,written by Brain Fox

### Shell Fundamentals

#### Command-line Editing Modes

The Command-line Editing Modes emulates the movement functions of two common txt editors,`emacs` and `vi` , in the case of the shell, the curosr's movement is being controlled

By Default, `bash` operates in `emacs` mode

##### Example Edit Commands

The following commands are very common while using the `emacs` mode

* `Ctrl-b`: Move backward by one character
* `Ctrl-f`: Move forward by one character
* `Ctrl-a`: Move to the beginning of the line
* `Ctrl-e`: Move to the end of the line
* `Ctrl-k`: Delete from the cursor forward
* `Ctrl-u`: Delete from the cursor backward
* `Ctrl-r`: Sarch the command history

## Enviroment variables

Enviroment variables are used to define values for often-used attributes of a user's shell

### $PATH

The most common, or most recongnized, environment variable is the `$PATH` variable. it defines the set of directoried that the shell can search to find a command. Without an explicit path provided when calling a command(i.e. `/bin/ps`), the shell will search the directoried listed in the `$PATH` variable until it finds the command.

To view the contents of the `$PATH` variable, use `echo` to print the variable's value:

        $ echo $PATH       
         /usr/local/bin/:/sbin:/bin/:/usr/sbin:/usr/bin:/usr/local/bin 

To list all of the shell's enviroment variables, use `env` command:

        $ env
        HOSTNAME=foobar
        SHELL=/bin/bash
        TERM=xterm
        HISTSIZE=1000
        USER=root
        PATH=/usr/local/sbin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:/usr/local/bin
        MAIL=/var/spool/mail/root
        PWD=/root/curriculum
        PS1=[\[\e[33;1m\]\t \[\e[31;1m\]\u\[\e[0m\]@\[\e[31;1m\]\h\[\e[0m\] \W\[\e[0m\]]#
        HISTCONTROL=ignoredups
        SHLVL=1
        SUDO_COMMAND=/bin/bash
        HOME=/root
        HISTTIMEFORMAT=[%Y-%m-%d %H:%M:%S]
        OLDPWD=/tmp

### Shell Profiles

Shell profiles are used to define a user's environment.

There are two types of profiles:
* Global profile(`/etc/profile`)
* User profile(`~/.bash_profile`)

`/etc/profile`

This is the global profile. Any environment variable set in this file applies to all users. Any script called from this file is executed for all users

`~/.bash_profile`

This is the user profile. Any environment variable set in this file applies to the user only. Any script called from this file is executed for the user only.

### Profile Precedence

It is possible to override settings from `/etc/profile` via `~/.bash_profile` as `~/.bash_profile` is executed after `/etc/profile`

## Special Environment Variables

Certain variables exist that provide useful information about what is happening in the enviroment, For example, it may be necessary to know the ID of a running process of the exist status of an executed command

### Process ID: `$$`

TO determine the PID of the current shell, firest run `ps` to find the PID, then run `echo $$` to confirm the PID
        
        $ ps -ef |grep bash 
         502 20440 20439   0 10:25PM ttys001    0:00.01 -bash                4006  31  0  2433436   1228 -      S                   0
         502 20447 20440   0 10:29PM ttys001    0:00.00 grep bash            4006  31  0  2432768    620 -  
        $ echo $$
          20440

### Background Process ID: `$!`

Occasionally, commands will need to be executed in the background(via the `&` operator). the PID of that process can be found using the `$!` variable.

        $ sleep 30 &
        [1] 20450
        $ echo $!
        20450

### Exit Status `$?`

When a command is executed, it always has an exit status. That status defines whether or not the command was successful. For success, the exit status is `0`, Non-zero values denote failure

        $ ls .bash_profile
        .bash_profile
        $ echo $?
        0

### History

The history is handy facility within `bash`; it stores all of the commands that the user has executed.

To see the history. simply type `history` and all of the stored commands will be dispalyed to the terminal. Similarity, run `cat ~/.bash_history` to see all stored commands

### Re-executing Commands

A benefit of storing command history is that the commands can be easily recalled. To execute the last command, type `!!`

        $ ls
        file1 file2 file3
        $ !!
        file1 file2 file3

To execute the nth command in history, run `!n` where `n` is the line number of the commands as found in the output of `history`:

        $ history | less
        1 ls -la
        2 ls -F 
        3 pwd

        $ !2
        ls -F
        file1 file2 dir1/

### Searching History

Finally, one can search the history by typing `Ctrl-r` followed by a string to match a command:

        $ (reverse-i-search)`ls': ls -F

### Job control

From time to time, it be necessary to manage commands running in the background. These are typically referred to as jobs. A command can be placed in the background via the `&` operatro. Use the `jobs` command to see the job in the background. To bring it back to foreground, run `fg`:

        $ sleep 30 &
        [1] 20969
        $ jobs
        [1]+ Running            sleep 30 &
        $fg
        sleep 30

While in the foreground, the job can be suspended via `Ctrl-z` and sent to the background once more using `bg`

### Multiple Jobs

It is possiable to have multiple jobs running in the background at the same time. Use the `jobs` command to track them via their job ID

        $ sleep 120 &
        [1] 21086
        $ sleep 240 &
        [2] 21087
        $ jobs
        [1]- Running    sleep 120 &
        [2]+ Running    sleep 240 &
        $ fg %1
        sleep 120
        $ kill %2