# different shells in linux

SHELL is a program which provides the interface between the user and an operating system. When the user logs in the OS starts a shell for the user.

Kernel controls all essential computer operations, provides the restriction to hardware access, coordinates all executing utilities, and manages resources between processes. Using the kernel only, a user can access utilities provided by the operating system.

### Types of Shells in Linux

#### The C Shell

Denoted as csh

* Bill Joy created it at the University of California at Berkeley. It incorporated features such as aliases and command history. It includes helpful programming features like built-in arithmetic and C-like expression syntax. In C shell:

```
Command full-path name is /bin/csh
Non-root user default prompt is hostname %
Root user default prompt is hostname # 
```

#### The Bourne Shell

Denoted as sh

* It was written by Steve Bourne at AT\&T Bell Labs. It is the original UNIX shell. It is faster and more preferred. It lacks features for interactive use like the ability to recall previous commands. It also lacks built-in arithmetic and logical expression handling. It is the default shell for Solaris OS. For the Bourne shell:

```
Command full-path name is /bin/sh and /sbin/sh
Non-root user default prompt is $
Root user default prompt is #
```

#### The Korn Shell

Denoted as ksh

* It was written by David Korn at AT\&T Bell Labs. It is a superset of the Bourne shell, so it supports everything in the Bourne shell. It has interactive features. It includes features like built-in arithmetic and C-like arrays, functions, and string-manipulation facilities. It is faster than C shell. It is compatible with scripts written for C shell. For the Korn shell:

```
Command full-path name is /bin/ksh
Non-root user default prompt is $
Root user default prompt is #
```

#### GNU Bourne-Again Shell

Denoted as bash

* It is compatible with the Bourne shell. It includes features from Korn and Bourne shell. For the GNU Bourne-Again shell:

```
Command full-path name is /bin/bash
Default prompt for a non-root user is bash-g.gg$ 
(g.gg indicates the shell version number like bash-3.50$)
Root user default prompt is bash-g.gg#
```

#### T Shell

Denoted as tsh

* It was originally developed for the Plan 9 operating system, but has since been ported to other systems, including Linux, FreeBSD, and macOS.

```
Command full-path name is /bin/tcsh
Default prompt for a non-root user is abhishekaslk(user):~>
Root user default prompt is root@abhishekaslk(user):~#.
```

#### Z Shell

Denoted by zsh

* Z Shell (zsh) was created by Paul Falstad in 1990 while he was a student at Princeton University. Z Shell is an extended version of the Bourne-Again Shell (bash), with additional features and capabilities.

```
Command full-path name is /bin/zsh
Default prompt for a non-root user is abhishekaslk(user)%
Root user default prompt is root@abhishekaslk(user):~#
```

<details>

<summary>How to check which shell I am using?</summary>

To check which shell you are currently using, you can use the following command:

```
echo $SHELL
```

This command will print the path to the shell executable. Another way is by using the `ps` command:

```
ps -p $$
```

This prints information about your shell process.

</details>

<details>

<summary>What are the features of the Bash shell?</summary>

Bash (Bourne-Again SHell) is one of the most feature-rich shells available. Some of its notable features include:

* Programmability: Includes advanced scripting capabilities, loops, conditionals, and user-defined functions.
* Job Control: Bash allows you to control jobs (processes) directly from the command line (pausing, resuming, and backgrounding processes).
* Command Line Editing: Interactive editing of command entries and command history features.
* Aliases and Shell Functions: Allows creating short names for commands or command sequences and more complex functions for custom operations.
* Environment Control: Provides extensive control over the environment that scripts and programs run in.
* Shell Expansions: Bash supports brace expansion, tilde expansion, parameter and variable expansion, command substitution, and arithmetic expansion.

</details>

<details>

<summary>How to switch between different shells in Linux?</summary>

To switch to a different shell during a session, simply type the name of the shell. For example, to switch to Zsh:

```
zsh
```

To change your default shell, use the `chsh` (change shell) command followed by the path to the new shell. You can find the path with `which`:

```
chsh -s $(which zsh)
```

This command will set Zsh as your default shell.

</details>

<details>

<summary>What are some lesser-known shells in Linux?</summary>

Beyond the commonly used shells, there are several lesser-known shells that provide unique features:

* mksh (MirBSD Korn Shell): A free implementation of the Korn Shell programming language and a successor to the public domain Korn shell (pdksh).
* yash (Yet Another SHell): A POSIX-compliant shell focusing on spec compliance and small footprint.
* oksh (OpenBSD Korn Shell): An OpenBSD-enhanced version of the Korn Shell, emphasizing simplicity, efficiency, and correctness.

</details>
