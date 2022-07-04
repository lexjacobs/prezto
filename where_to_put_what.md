# What should/shouldn't go in .zshenv, .zshrc, .zlogin, .zprofile, .zlogout?

## _I'm looking for guidelines on what one should and should not include in the various startup files for zsh. I understand the order of sourcing of these files, and the conditions under which they are sourced, but it is still not clear to me what should go in each._
---
Here is a non-exhaustive list, in execution-order, of what each file tends to contain:

.zshenv is always sourced. It often contains exported variables that should be available to other programs. For example, $PATH, $EDITOR, and $PAGER are often set in .zshenv. Also, you can set $ZDOTDIR in .zshenv to specify an alternative location for the rest of your zsh configuration.

.zprofile is for login shells. It is basically the same as .zlogin except that it's sourced before .zshrc whereas .zlogin is sourced after .zshrc. According to the zsh documentation, ".zprofile is meant as an alternative to .zlogin for ksh fans; the two are not intended to be used together, although this could certainly be done if desired."

.zshrc is for interactive shells. You set options for the interactive shell there with the setopt and unsetopt commands. You can also load shell modules, set your history options, change your prompt, set up zle and completion, et cetera. You also set any variables that are only used in the interactive shell (e.g. $LS_COLORS).

.zlogin is for login shells. It is sourced on the start of a login shell but after .zshrc, if the shell is also interactive. This file is often used to start X using startx. Some systems start X on boot, so this file is not always very useful.

.zlogout is sometimes used to clear and reset the terminal. It is called when exiting, not when opening.
You should go through the configuration files of random Github users to get a better idea of what each file should contain.

---

Be aware when setting $PATH in .zshenv, various other files all are sourced after this file that will override this value. See zsh.org/mla/users/2003/msg00600.html. –
Beau Barker
98

Just for my own notes / confirmation and to help anybody else, the ultimate order is .zshenv → [.zprofile if login] → [.zshrc if interactive] → [.zlogin if login] → [.zlogout sometimes]. –
Gabriel L.
8

re: setting $PATH in .zshenv - macOS, as of Big Sur and at least a few versions prior - overwrites any changes to $PATH when /etc/zprofile is executed (after .zshenv). –
Lyndsy Simon
1

@BeauBarker should $PATH be located in .zshrc? –
alper

@alper GitHub user thoughtbot's popular dotfiles do indeed recommend setting the $PATH in .zshrc. –
Paul Razvan Berg

134

---

### Here a list of what each file should/shouldn't contain, in my opinion:

.zshenv

[Read every time]

This file is always sourced, so it should set environment variables which need to be updated frequently. PATH (or its associated counterpart path) is a good example because you probably don't want to restart your whole session to make it update. By setting it in that file, reopening a terminal emulator will start a new Zsh instance with the PATH value updated.

But be aware that this file is read even when Zsh is launched to run a single command (with the -c option), even by another tool like make. You should be very careful to not modify the default behavior of standard commands because it may break some tools (by setting aliases for example).

.zprofile
[Read at login]

I personally treat that file like .zshenv but for commands and variables which should be set once or which don't need to be updated frequently:

environment variables to configure tools (flags for compilation, data folder location, etc.)
configuration which execute commands (like SCONSFLAGS="--jobs=$(( $(nproc) - 1 ))") as it may take some time to execute.
If you modify this file, you can apply the configuration updates by running a login shell:

exec zsh --login
.zshrc
[Read when interactive]

I put here everything needed only for interactive usage:

prompt,
command completion,
command correction,
command suggestion,
command highlighting,
output coloring,
aliases,
key bindings,
commands history management,
other miscellaneous interactive tools (auto_cd, manydots-magic)...
.zlogin
[Read at login]

This file is like .zprofile, but is read after .zshrc. You can consider the shell to be fully set up at .zlogin execution time

So, I use it to launch external commands which do not modify shell behaviors (e.g. a login manager).

.zlogout
[Read at logout][Within login shell]

Here, you can clear your terminal or any other resource which was setup at login.

How I choose where to put a setting
if it is needed by a command run non-interactively: .zshenv
if it should be updated on each new shell: .zshenv
if it runs a command which may take some time to complete: .zprofile
if it is related to interactive usage: .zshrc
if it is a command to be run when the shell is fully setup: .zlogin
if it releases a resource acquired at login: .zlogout
