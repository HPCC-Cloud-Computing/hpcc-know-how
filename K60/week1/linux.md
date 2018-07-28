# Linux overview

## Basic commands

Command|Description
-|-
logname   |  Print the name of current user
hostname  |  Print hostname
clear  |  Clear screen
who  |  Prints information about users who are currently logged in
exit  | Cause normal process termination
passwd  |  Change password
dare  |  Print date and time
mkdir  |  Make new directories
rmdir  |  Remove directories
cd  |  Change working directory
pwd  | Print current directory
cp  |  Copy files and directories
rm  |  Remove files and directories
ps  |  Report a snapshot of the current processes
ssh  |  Run a ssh client program to execute a command on a remote host

## Linux file system

### Common directories

*   / : root directory
    -   /bin:  Contain application binary files
    -   /boot: OS core directory
    -   /etc:  Configuration directory
    -   /dev:  Device directory
    -   /home: User data directory
    -   /lib:  System library
    -   /usr:  Application directory
    -   /var:  Contain variable data files

### Linux files vs. Windows files

#### Linux specification

Everything is under the file view point.

All system resources behave like a file.

No extensions in file name.

No drive letters.

'/' instead of '\'.

### Special links and directory

*   / : root directory
*   ~/: home directory
*   . : Current directory
*   ..: Parent directory

### Directory manipulation commands

*   pwd
*   cd
*   ls
*   mkdir
*   rmdir

### File types

*   -: normal file
*   d: directory
*   b: block
*   c: character
*   l: link
*   m: shared memory
*   p: pipeline

### File operations

*   `cp`
*   `mv`
*   `rm`
*   `cat`  : concatnate file to stdout
*   `more` : paging through text one screenful at a time
*   `less` : read one line at a time
*   `tail` : read the first n line of a file
*   `head` : read the last n line of a file
*   `touch`: create empty file
*   `echo` > [filename]

### Inode

Each file assocciates with an inode in which metadata of the file (owner, last access, modifcation, permission,...) is stored.

Virtual addresses of each data block of each file are also stored in an inode.

Directories are lists of names assigned to inode.

### Links

A link is basicly a reference to an inode, making a link to a file (inode) is making a shortcut to that file.

#### Hard links

**Syntax**

        $ ln filename hardlink

**Note**

*   You cannot make a hard link point to a directory, you also can't make a hard link point to an file that doesn't exist.
*   When you create a file, you create an inode and a hard link point to it.
*   Removing a file is removing a hard link point to its inode.
*   By removing the last hard link point to an inode, its relevant blocks are removed as well.

#### Symbolic links

**Syntax**

        $ ln -s filename symboliclink

**Note**

*   You can make a symblic link that points to anything, even non-existed files.
*   Remove a symbolic link doesn't effect its inode.

### Searching for files

#### Syntax

        $ find directory expressions

**Conditions**

*   -name filename
*   -perm permission
*   -type d/f/...
*   -size
*   -time

**Operations on files**

*   -print
*   -exec command

## User account and file permissions

### Users and group users

*   User information are stored in `/etc/passwd`.
*   Password information are stored in `/etc/shadow`.
*   Group information are stored in `/etc/group`.
*   Group password and permissions are stored in `/etc/gshadow`.

### File permissions

#### Three types of permission:
*   **r** : read.
*   **w** : write.
*   **x** : execute.

#### Three types of user:
*   **u** : unique owner of a file.
*   **g** : group owner of a file.
*   **o** : other, none of these above.

Each user has a set of (r, w, x) rights for each file.

#### Special permissions for excutable files

*    **s** (set-uid or set-gid) : Program can only be executed by its user or group owner.
*    **t** (sticky bit): Allocate memory once for the program.

#### Modify file permissions

**Syntax**

        $chmod <mode> <files>

`mode` is represented in four tripplet of bits, the least significant bit is in the right most and vice versa:
*   (set_uid, set-gid, sticky)
*   user (r, w, x)
*   group (r, w, x)
*   other (r, w, x)

_Example:_

        $ chmod 6711 filename

--> **set-uid**, **set-gid**, **rwx** for user, **x** for group and **x** for other.

#### Default file permission

        $ umask <mask>

_Example_:

        $ umask 022

--> u|g|o : rwx|r-x|r-x

#### Change file owner

Use `chown` and `chgrp`, only for root user.

## Process management

### System processes

Belong to root, normally are daemon, serve general purposes.

_Example:_ `ipsched`, `cron`, `inetd`

### User processes

Belong to users, executed in shell, managed by terminal.

_Example:_ `cp`, `vi`, `man`

### Process management commands

#### Show current processes

`ps` : show user's processes.

`ps aux` : show all processes.

### Process state

`Running` + `Ctrl-Z` --> `Stop`

`Running` + `Ctrl-C` --> `Terminate`

`Stop` + `kill` --> `Terminate`

`Stop` + `fg` --> `Running`

### Kill a process

`kill -9 PID` : kill process with given PID.

`killall -9 COMMAND`: kill all processes with given command name.

### Process preference

*   Default preference is **0**, with range from **-19** to **19**.
*   Only root user can decrease preference of a process.
*   Any user can increase its own processes' preference.
*   `nice` command sets process initial preference.

        nice [-n value] [Command [args]]

*   `renice` command sets command preference while running.

Use `top` command to see processes with % CPU and RAM usage in descending order.

## Linux standard streams

### Standard streams

Each process has its own `stdin`, `stdout` and `stderr`.

`/dev/null` is special file that is used to trash any data directed to it.



### Stream redirection

#### Overwrite

*   `>` : standard output
*   `<` : standard input
*   `2>`: standard error

#### Append

*   `>>` : standard output
*   `<<` : standard input
*   `2>>`: standard error

### Pipes

Feed output data of command1 to input stream of command2.

        command1 | command2

### Filters

Alter piped redirection and output.

`find` - Returns files with filenames that match the argument passed to find.

`grep` - Returns text that matches the string pattern passed to grep.

`tee` - Redirects standard input to both standard output and one or more files.

`tr` - Finds-and-replaces one string with another.

`wc` - Counts characters, lines, and words.
