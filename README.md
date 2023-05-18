# 0X16. C - Simple Shell :shell:

A simple UNIX command interpreter written as part of the low-level programming for the ALX SE program. The project is given the name myshell

## Description :speech_balloon:

**myshell** is a simple UNIX command language interpreter that reads commands from either a file or standard input and executes them.

### Invocation :running:

Usage: **myshell** [filename]

To invoke **myshell**, compile all `.c` files in the repository and run the resulting executable:

```
gcc *.c -o myshell
./myshell
```

**myshell** can be invoked both interactively and non-interactively. If **myshell** is invoked with standard input not connected to a terminal, it reads and executes received commands in order.

Example:
```
$ echo "echo 'hello'" | ./myshell
'hello'
$
```

If **myShell** is invoked with standard input connected to a terminal (determined by [isatty](https://linux.die.net/man/3/isatty)(3)), an *interactive* shell is opened. When executing interactively, **myShell** displays the prompt `$ ` when it is ready to read a command.

Example:
```
$./myshell
$
```

Alternatively, if command line arguments are supplied upon invocation, **myshell** treats the first argument as a file from which to read commands. The supplied file should contain one command per line. **myShell** runs each of the commands contained in the file in order before exiting.

Example:
```
$ cat test
echo 'hello'
$ ./myshell test
'hello'
$
```

### Environment :deciduous_tree:

Upon invocation, **myShell** receives and copies the environment of the parent process in which it was executed. This environment is an array of *name-value* strings describing variables in the format *NAME=VALUE*. A few key environmental variables are:

#### HOME
The home directory of the current user and the default directory argument for the **cd** builtin command.

```
$ echo "echo $HOME" | ./myshell
/home/vagrant
```

#### PWD
The current working directory as set by the **cd** command.

```
$ echo "echo $PWD" | ./myshell
/home/vagrant/Cinnamon/simple_shell
```

#### OLDPWD
The previous working directory as set by the **cd** command.

```
$ echo "echo $OLDPWD" | ./myshell
/home/vagrant/Cinnamon/printf
```

#### PATH
A colon-separated list of directories in which the shell looks for commands. A null directory name in the path (represented by any of two adjacent colons, an initial colon, or a trailing colon) indicates the current directory.

```
$ echo "echo $PATH" | ./myshell
/home/vagrant/.cargo/bin:/home/vagrant/.local/bin:/home/vagrant/.rbenv/plugins/ruby-build/bin:/home/vagrant/.rbenv/shims:/home/vagrant/.rbenv/bin:/home/vagrant/.nvm/versions/node/v10.15.3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/home/vagrant/.cargo/bin:/home/vagrant/workflow:/home/vagrant/.local/bin
```

### Command Execution :hocho:

After receiving a command, **myShell** tokenizes it into words using `" "` as a delimiter. The first word is considered the command and all remaining words are considered arguments to that command. **myShell** then proceeds with the following actions:
1. If the first character of the command is neither a slash (`\`) nor dot (`.`), the shell searches for it in the list of shell builtins. If there exists a builtin by that name, the builtin is invoked.
2. If the first character of the command is none of a slash (`\`), dot (`.`), nor builtin, **shellby** searches each element of the **PATH** environmental variable for a directory containing an executable file by that name.
3. If the first character of the command is a slash (`\`) or dot (`.`) or either of the above searches was successful, the shell executes the named program with any remaining given arguments in a separate execution environment.

### Exit Status :wave:

**myShell** returns the exit status of the last command executed, with zero indicating success and non-zero indicating failure.

If a command is not found, the return status is `127`; if a command is found but is not executable, the return status is 126.

All builtins return zero on success and one or two on incorrect usage (indicated by a corresponding error message).

### Signals :exclamation:

While running in interactive mode, **myshell** ignores the keyboard input `Ctrl+c`. Alternatively, an input of end-of-file (`Ctrl+d`) will exit the program.

User hits `Ctrl+d` in the third line.
```
$ ./myshell
$ ^C
$ ^C
$
```

### Variable Replacement :heavy_dollar_sign:

**myShell** interprets the `$` character for variable replacement.

#### $ENV_VARIABLE
`ENV_VARIABLE` is substituted with its value.

Example:
```
$ echo "echo $PWD" | ./myshell
/home/vagrant/holberton/simple_shell
```

#### $?
`?` is substitued with the return value of the last program executed.

Example:
```
$ echo "echo $?" | ./myshell
0
```

#### $$
The second `$` is substitued with the current process ID.

Example:
```
$ echo "echo $$" | ./myshell
6494
```

### Comments :hash:

**myShell** ignores all words and characters preceeded by a `#` character on a line.

Example:
```
$ echo "echo 'hello' #this will be ignored!" | ./myshell
'hello'
```

### Operators :guitar:

**myShell** specially interprets the following operator characters:

#### ; - Command separator
Commands separated by a `;` are executed sequentially.

Example:
```
$ echo "echo 'hello' ; echo 'world'" | ./myshell
'hello'
'world'
```

#### && - AND logical operator
`command1 && command2`: `command2` is executed if, and only if, `command1` returns an exit status of zero.

Example:
```
$ echo "error! && echo 'hello'" | ./myshell
./myshell: 1: error!: not found
$ echo "echo 'all good' && echo 'hello'" | ./myshell
'all good'
'hello'
```

#### || - OR logical operator
`command1 || command2`: `command2` is executed if, and only if, `command1` returns a non-zero exit status.

Example:
```
$ echo "error! || echo 'but still runs'" | ./myshell
./myshell: 1: error!: not found
'but still runs'
```

The operators `&&` and `||` have equal precedence, followed by `;`.

### myShell Builtin Commands :nut_and_bolt:

#### cd
  * Usage: `cd [DIRECTORY]`
  * Changes the current directory of the process to `DIRECTORY`.
  * If no argument is given, the command is interpreted as `cd $HOME`.
  * If the argument `-` is given, the command is interpreted as `cd $OLDPWD` and the pathname of the new working directory is printed to standad output.
  * If the argument, `--` is given, the command is interpreted as `cd $OLDPWD` but the pathname of the new working directory is not printed.
  * The environment variables `PWD` and `OLDPWD` are updated after a change of directory.

Example:
```
$ ./myShell
$ pwd
/home/vagrant/Cinnamon/simple_shell
$ cd ../
$ pwd
/home/vagrant/Cinnamon
$ cd -
$ pwd
/home/vagrant/Cinnamon/simple_shell
```

#### alias
  * Usage: `alias [NAME[='VALUE'] ...]`
  * Handles aliases.
  * `alias`: Prints a list of all aliases, one per line, in the form `NAME='VALUE'`.
  * `alias NAME [NAME2 ...]`: Prints the aliases `NAME`, `NAME2`, etc. one per line, in the form `NAME='VALUE'`.
  * `alias NAME='VALUE' [...]`: Defines an alias for each `NAME` whose `VALUE` is given. If `name` is already an alias, its value is replaced with `VALUE`.

Example:
```
$ ./myshell
$ alias show=ls
$ show
AUTHORS            builtinsHelp02.c  getline.c        locate_file.c     simple_shell_man
builtin_aliases.c  env.c             helpers01.c      main.c            split.c
builtins.c         error01.c         helpers.c        proc_file_comm.c  str_functions01.c
builtins_env.c     error02.c         input_helpers.c  README.md         str_functions02.c
builtinsHelp01.c   errors.c          linkedlist.c     shell.h
```

#### exit
  * Usage: `exit [STATUS]`
  * Exits the shell.
  * The `STATUS` argument is the integer used to exit the shell.
  * If no argument is given, the command is interpreted as `exit 0`.

Example:
```
$ ./myshell
$ exit
```

#### env
  * Usage: `env`
  * Prints the current environment.

Example:
```
$ ./myshell
$ env
NVM_DIR=/home/vagrant/.nvm
...
```

#### setenv
  * Usage: `setenv [VARIABLE] [VALUE]`
  * Initializes a new environment variable, or modifies an existing one.
  * Upon failure, prints a message to `stderr`.

Example:
```
$ ./myshell
$ setenv NAME ChezDev
$ echo $NAME
ChezDev
```

#### unsetenv
  * Usage: `unsetenv [VARIABLE]`
  * Removes an environmental variable.
  * Upon failure, prints a message to `stderr`.

Example:
```
$ ./myshell
$ setenv NAME ChezDev
$ unsetenv NAME
$ echo $NAME

$
```

## Authors :black_nib:

* Hillary Cheserek <[Hiluhree](https://github.com/Hiluhree)>

## Acknowledgements :pray:

**myShell** emulates basic functionality of the **sh** shell. This README borrows form the Linux man pages [sh(1)](https://linux.die.net/man/1/sh) and [dash(1)](https://linux.die.net/man/1/dash).

This project was written as part of the curriculum for ALX SE program. ALX SE prgram is an online-based full-stack software engineering program that prepares students for careers in the tech industry using project-based peer learning. For more information, visit [this link](https://www.alxafrica.com/).

<p align="center">
  <img src="http://www.alxafrica.com/alx-logo.png" alt="ALX logo">
</p>
