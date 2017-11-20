---
title: linux-commands
date: 2017-07-18 21:33:50
categories: [后台开发, Linux]
tags: [Linux, Unix]
---
# Concepts
Unix is multi-user, multi-process, mutli-access operating system.

Each command is executed as a new process, and is the child of the process which invoked it.

## Kernel Responsibilities
* Process Management
* Memory Management
* File System Management
* Device Management

# Basic Commands
## `who` vs `whoami`
`who` it gives the list of all users currently logged in to the machine
`whoami` let you know the current user who is in the shell.
## Keyboard Control

| options | meaning |
| ------------- |-----:|
| ^C | interrupt command |
| ^Z | suspend command |
| ^D | end of file |
| ^S | suspend output |
| ^Q | continue output |
| ^H (^?) | delete last character |
| ^W | delete last word |
| ^U | delete line |

# File System
## commands
`ls` - list directory contents
`ls -F` - append indicator (one of */=>@|) to entries ('/' means directory)
`ls -l` - use a long listing format
`ls -i` - print the index number of each file
`wc` - print newline, word, and byte counts for each file
`cat` - concatenate files and print on the standard output
`mkdir` - creates directories
`rmdir` - removes directories
`cp` - copies files and directories around the file system
(P.S. wildcards cannot be used for names which don't exist)
`mv` - move files and sub-directories
`rm` - deletes files and direcotry structures
`rm -i` requests confirmation before deleting files
`head` - displays the first lines of a file
`tail` - displays the last lines of a file
(e.g. head -4 userlist - display the first 4 lines of userlist)
(e.g. tail -3 userlist - display the last 3 lines of userlist)
`grep` - searches files for strings

## Hierarchical Structure
Directories can be viewed as branches and files as leaves.

## Linking Files
`ln` - create links
`ln -s` -  make symbolic links instead of hard links
(P.S.Symbolic links source can be files or directories.Hard links source only files.类似指针)

## Searching Files
`grep` options

| option | meaning |
| ------------- |-----:|
| -c | display count of matching lines |
| -i | case insensitive |
| -l | list names of files containing matching lines |
| -n | precede each line by its line number |
| -v | only display lines that do *not match* |
| -w | search for the expression as a word |

can use Regex expressions for strings.

## Quoting
Quoting allows characters to be hidden from the shell.

    e.g. $ echo hello      world
    hello world
    $ echo "hello   world"
    hello   world

## Sorting Files
`sort` - organises files into alpha-numeric order
`sort -n` - sort on numeric value
`sort -r` - reverse the sort
`sort +#` - skip # fields to start of sort key
(e.g. `sort +3n` means from 3rd fields to start sort on numeric value)

## Comparing Files
`diff` - displays the differences between two files
`diff -e` - displays the differences in second files

## Cutting Fields
`cut` - removes selected fields from each line of the file
`cut -d` -  use DELIM instead of TAB for field delimiter.自定义分隔符，通常是制表符。
`cut -f`-  f means fields. select only these fields;  also print anyline that contains  no delimiter character, unless the -s option is specified
`cut -b` - select bypes

## Duplicate Lines
`uniq` - removes duplicate adjacent lines from files
`uniq -c` - count number of duplicate line before files lines
`uniq -d` - only print repeated lines
`uniq -u` - only print non-repeated lines

## Transliterating Files
`tr` - translate characters from input to output.
(Default input is STDIN. Default output is STDOUT. )
`tr SET1 [SET2]` - translate SET1 in input to SET2.

## Printing Files
`lpr` - send files to a printer
`lpq` - queries the state of the print queue
`lprm` - dequeues jobs from the print queue.remove the jobs.

## Searching for Files
`find` - searches any part of the file system.

* it can apply any Unix command to the files found
* it can print the file name found
* it can search for files based on several file attributes

| options | meaning |
| ------------- | -----:|
| -name | filename pattern |
| -user | user name |
| -group | group name |
| -size | [+/-] size in blocks |
| -perm | [-] octal number |
| -atime | [+/-] days |
| -mtime | [+/-] days |
| -ctime | [+/-] days |
| -type | [d/f/l] |
| -inum | i-node number |

# File Access Control
## Permissions
### Access rights for files

| rights | meaning | meaning |
| ------------- |:-------------:| -----:|
| r | read | read contens |
| w | write | update and implies delete |
| x | execute | attempt to run program |
| s | set ID | change the UID or GID of process |

### Access rights for directories

| rights | meaning | meaning |
| ------------- |:-------------:| -----:|
| r | list | list contens |
| w | write | create and delete any files |
| x | search | search directory(cd) |
| t | sticky | control write access to directories |

use `ls -l` to display access rights. The order is type, user, group, others.
e.g. drwxrw-r--

| type | user | group | others |
| ------------- |:-------------:|:-------------:| -----:|
| d | rwx | rw- | r-- |

## Setting Permissions
`chmod` - change permissions
`chomd -R` -  enables recursion through a directory(递归遍历)

The permission mode is set using a symbolic notation and a numeric notation.

### Symbolic Notation
e.g. `chmod go+r aFile` means group and others add r access rights.

### Numeric Notation
treats the permissions as a bit pattern.
e.g. 

| r | w | x | total |
| ------------- |:-------------:| -----:|
| 4 | 2 | 1 | 7 means rwx |
| 4 | 0 | 1 | 5 means rx |

## Default Permissions
`umask` - defines the default permission bits. in numeric notation to show permission.

# Editing Files

| ------------- | -----:|
| `ed` | interactive, buffered, line oriented |
| `vi` | interactive, buffered, screen oriented |
| `sed` | non-interactive, non-buffered, stream oriented |

## ed

`ed [-p prompt] [filename]`

`[addr,[addr]]<character command> [parameters] `

| command | meaning |
| ------------- | -----:|
| i | insert |
| a | append |
| c | change |
| d | delete |
| p | print |
| w | write to file |
| r | read from file |
| s/RE/RS/g | substitute RE for RS |
| t addr | transfer to addr |
| q | quit |
| Q | really quit |

## vi
`vi [-r] [filename]`

### command mode

| command | meaning |
| ------------- | -----:|
| h | cursor left |
| j | cursor down |
| k | cursor up |
| l | cursor right|
| w | word forward |
| b | word backward |
| ^u | page up |
| ^d | page down |
| ^ | start of line |
| $ | end of line |
| G | go to line |
| i | insert before cursor |
| I | insert at start of line |
| a | insert after cursor |
| A | insert at the end of line |
| r | replace character |
| R | overwrite rest of line |
| o | open line below |
| O | open line above |
| x | delete current character |
| X | delete character before cursor |
| d | delete text |
| D | delete the rest of the line |
| y | yank text into buffer |
| p | paste buffer before |
| P | paste buffer after |
| / | search forward for RE |
| ? | search back for RE |
| n | repeat search |
| . | repeat last change |
| u | undo last change |
| N | reverse search |
| ZZ | write file then quit |

### ex Mode
allows you to enter ex(ed) commands

| command | meaning |
| ------------- | -----:|
| :wq | write file then quit |
| :x | write file then quit |
| :q! | force quit without save |
| :q | quit |

# Process Commands
`more` - used to paginate files.
`more` is invoked as a new process by the shell.
`cd` and `pwd` are commands to the shell itself and don't give rise to new processes.

# The Shell
Login shell is invoked as the user logs in, dies when the user logs out.
Wildcards be used to generate filenames.

| wildcards | meaning |
| ------------- | -----:|
| * | any number of characters |
| ? | any single character |
| [ab] | a or b or specified range |

## Shell I/O Redirection
Standard I/O is STDIN, STDOUT, STDERR(producing error messages)

| symbol | meaning |
| ------------- | -----:|
| `>` | redirect output |
| `>>` | append to existing file |
| `<` | redirect input |
| `<<` | redirect STDIN from command itself |
| `|` | pipeline commands |

## Background Jobs
use `&` to put jobs into the background.
`jobs` - list all jobs in the shell

`bg %n` - move a job to the background
`fg %n` - move a job into the foreground
`stop %n` - stop a background job - a stopped job can be restarted
`^Z` - stop a foreground job

%n -- n is the number in the square brackets from the `jobs` command.

## new processes
The parent process waits until the child process ends.  
The child process inherits the parent attributes.  
When the child process dies, its exit code is sent to its parent, waking the parent process up.

## Shell Variables
`set` - displays local variables
`env` - displays environment variables
`aliases` - create alternative commands

## Shell Program

$0 - Command nae
$# - Number of arguments
$*, $@ - All of the arguments
"$*" - one argument of all parameters from $1, like "$1 $2 … $n"
"$@" - All aruments from $1, like "$1" "$2" … "$n" 
`shift` - moves arguments down to access others (弹出一个参数)
`read` - reads from standard input

	read name address
	echo goodbye $name of $address

`test` - evaluates general conditions. succeed return 0, otherwise non zero.
`expr` - evaluate expressions, cuz shell does not understand numbers, each argument must be separated by spaces.

## sed
* A non-interactive, stream-oriented editor.
* works on lines of text
* input flows through the program and is directed to standard output

| option | meaning |
| -f | allows a sed script file to be specified |
| -e | Precedes each edit when multiple edits are defined |
| -n | Suppress the default output |