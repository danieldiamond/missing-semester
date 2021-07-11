# Missing Semester 2020
Notes from the [MIT Missing Semester 2020 Course](https://missing.csail.mit.edu/2020/) covering computing ecosystem literacy.

1. [Course overview + the shell](#1-course-overview---the-shell)
2. [Shell Tools and Scripting](#2-shell-tools-and-scripting)
    - [STDOUT, STDERR and Return Code](#stdout--stderr-and-return-code)
    - [Command and Process Substitution](#command-and-process-substitution)
    - [Special Variables](#special-variables)
    - [Other Commands](#other-commands)
3. [Editors (Vim)](#3-editors--vim-)
    - [Modal editing](#modal-editing)
    - [Terminology](#terminology)
    - [Command Mode](#command-mode)
    - [Normal Mode](#normal-mode)
    - [Selection Mode](#selection-mode)
    - [Edits (normal mode)](#edits--normal-mode-)
    - [Modifiers](#modifiers)
    - [Customize and Extend vim](#customize-and-extend-vim)
4. [Data Wrangling](#4-data-wrangling)
    - [sed (stream editor)](#sed--stream-editor-)
    - [Regex Recap:](#regex-recap-)
    - [Additional CLI commands:](#additional-cli-commands-)
    - [awk](#awk)
5. [Command Line Environment](#5-command-line-environment)
    - [Job Control](#job-control)
    - [Pausing and backgrounding processes](#pausing-and-backgrounding-processes)
    - [Terminal Multiplexers](#terminal-multiplexers)
    - [Aliases](#aliases)
    - [Dotfiles and Portability](#dotfiles-and-portability)
    - [Remote Machines](#remote-machines)
    - [Port Forwarding](#port-forwarding)
    - [SSH Configurations](#ssh-configurations)
6. [Version Control (Git)](#6-version-control--git-)
    - [Objects and content-addressing](#objects-and-content-addressing)
    - [References](#references)
    - [Advanced Git](#advanced-git)
7. [Debugging and Profiling](#7-debugging-and-profiling)
    - [system log](#system-log)
    - [Debugger](#debugger)
    - [(CPU) Profiling (Tracing vs Sampling)](#-cpu--profiling--tracing-vs-sampling-)
        + [A note on Time](#a-note-on-time)
    - [Resource Monitoring](#resource-monitoring)
8. [Metaprogramming](#8-metaprogramming)
    - [Build Systems](#build-systems)
    - [Dependency Management](#dependency-management)
    - [Versioning](#versioning)
        + [lock file](#lock-file)
        + [vendoring](#vendoring)
    - [Continuous integration systems](#continuous-integration-systems)
    - [Testing](#testing)
9. [Security & Cryptography](#9-security---cryptography)
    - [Entropy](#entropy)
    - [Hash Functions](#hash-functions)
        + [Hash Properties](#hash-properties)
        + [Applications](#applications)
    - [Key Derivation Functions (KDF)](#key-derivation-functions--kdf-)
        + [Applications](#applications-1)
    - [Symmetric Cryptography](#symmetric-cryptography)
        + [Properties](#properties)
        + [Applications](#applications-2)
    - [Asymmetric Cryptography](#asymmetric-cryptography)
        + [Applications](#applications-3)
        + [Key Distribution](#key-distribution)
        + [Case Studies](#case-studies)
10. [Random Topics](#10-random-topics)
    - [Daemons](#daemons)
    - [Filesystems in User Space (FUSE)](#filesystems-in-user-space--fuse-)
    - [Hammerspoon (Automation for MacOS)](#hammerspoon--automation-for-macos-)
    - [Where are programs located?](#where-are-programs-located-)


## 1. Course overview + the shell
The basics

## 2. Shell Tools and Scripting
The basics
- shell scripting optimized for performing shell related tasks
- man or [tldr](https://tldr.sh/)
- ' delimited strings are literal strings and will not substitute variable values. " delimited strings will
### STDOUT, STDERR and Return Code
commands return output via STDOUT, errors via STDERR and a Return Code.
```
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

false ; echo "This will always run"
# This will always run
```
### Command and Process Substitution
get output of a command as a variable --> _command substitution_ i.e. $(CMD)
- e.g. `for file in $(ls)` --> first call ls then iterate over files
put output of command into a temporary file (or pipe to another command) --> _process substitution_ i.e. <( CMD )
 - e.g. `diff <(ls foo) <(ls bar)` --> differences between files in dirs foo and bar
### Special Variables
[bash special variables] (https://www.tldp.org/LDP/abs/html/special-chars.html)
- $0 - Name of the script
- $1 to $9 - Arguments to the script. $1 is the first argument and so on.
- $@ - All the arguments
- $# - Number of arguments
- $? - Return code of the previous command
- $$ - Process Identification number for the current script
- !! - Entire last command, including arguments. A common pattern is to execute a command only for it to fail due to missing permissions, then you can quickly execute it with sudo by doing sudo !!
- $_ - Last argument from the last command. If you are in an interactive shell, you can also quickly get this value by typing Esc followed by .

e.g. iterate through the arguments we provide, grep for the string foobar and append it to the file as a comment if it’s not found
```
#!/bin/bash

# command substitution
echo "Starting program at $(date)" # Date will be substituted

# special variables: $0 program name, $$ pid of script
echo "Running program $0 with $# arguments with pid $$"

# for file in all arguments
for file in $@; do

    # grep for foobar in file and redirect STDOUT and STDERR to null register 
    grep foobar $file > /dev/null 2> /dev/null
    # if pattern is not found, grep has exit status 1 ($? is previous code)
    if [[ $? -ne 0 ]]; then
        echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done
```
### Other Commands
- [test command](http://man7.org/linux/man-pages/man1/test.1.html)
```
       INTEGER1 -ne INTEGER2
              INTEGER1 is not equal to INTEGER2
```
- globbing
    - ? or * --> wildcards one or many
    - {a..d} automatically expand
        - e.g. `touch {foo,bar}/{a..j}` --> files foo/a, foo/b, ... foo/h, bar/a, bar/b, ...

- bash scripts in other languages
    - NOTE: `#!/usr/bin/env python` preferred over `#!/usr/local/bin/python`
```
#!/usr/local/bin/python
import sys
for arg in reversed(sys.argv[1:]):
    print(arg)
```

- find files
```
find . -name src -type d # Find all directories named src
find . -path '**/test/**/*.py' -type f # Find all python files that have a folder named test in their path
find . -mtime -1 # Find all files modified in the last day
find . -size +500k -size -10M -name '*.tar.gz' # Find all zip files with size in range 500k to 10M
```
- find and *action*
```
find . -name '*.tmp' -exec rm {} \; # Delete all files with .tmp extension
find . -name '*.png' -exec convert {} {.}.jpg \; # Find all PNG files and convert them to JPG
```
- [find vs locate](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)
- [ripgrep (rg)](https://github.com/BurntSushi/ripgrep)
```
rg -t py 'import requests' # Find all python files where I used the requests library
rg -u --files-without-match "^#!" # Find all files (including hidden files) without a shebang line
rg foo -A 5 # Find all matches of foo and print the following 5 lines
rg --stats PATTERN # Print statistics of matches (# of matched lines and files )
```
- move on from bash --> `zsh`, `fish` etc.
    - enable <b>history-based autosuggestions</b>
    - organizing dotfiles

## 3. Editors (Vim)
- vim: command-line-based editor: modal editor
### Modal editing
Modes:
- <b>Normal</b>: for moving around a file and making edits
- <b>Insert</b>: for inserting text
- <b>Replace</b>: for replacing text
- <b>Visual (plain, line, or block) mode</b>: for selecting blocks of text
- <b>Command-line</b>: for running a command

### Terminology
- buffers ~ files
- session ~ number of tabs
- windows ~ split panes (each tab can have multiple windows)
    - each window shows a single buffer
- NOTE: No 1-to-1 correspondence between buffers and windows; windows are merely views. A given buffer may be open in multiple windows, even within the same tab. This can be quite handy, for example, to view two different parts of a file at the same time.
- By default, Vim opens with a single tab, which contains a single window.

[map Caps Lock to <Esc>](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)

### Command Mode 
```
- :q quit (close window)
- :w save (“write”)
- :wq save and quit
- :e {name of file} open file for editing
- :ls show open buffers
- :help {topic} open help
    - :help :w opens help for the :w command
    - :help w opens help for the w movement
```
### Normal Mode
```
- Basic movement: hjkl (left, down, up, right)
- Words: w (next word), b (beginning of word), e (end of word)
- Lines: 0 (beginning of line), ^ (first non-blank character), $ (end of line)
- Screen: H (top of screen), M (middle of screen), L (bottom of screen)
- Scroll: Ctrl-u (up), Ctrl-d (down)
- File: gg (beginning of file), G (end of file)
- Line numbers: :{number}<CR> or {number}G (line {number})
- Misc: % (corresponding item)
- Find: f{character}, t{character}, F{character}, T{character}
- find/to forward/backward {character} on the current line
, / ; for navigating matches
- Search: /{regex}, n / N for navigating matches
```
### Selection Mode
- visual
- visual line
- visual block

### Edits (normal mode)
- i enter insert mode
- o / O insert line below / above
- d{motion} delete {motion} e.g:
    - dw is delete word
    - d$ is delete to end of line
    - d0 is delete to beginning of line
- c{motion} change {motion} e.g:
    - cw is change word
- x delete character (equal do dl)
- s substitute character (equal to xi)
- visual mode + manipulation
    - select text, d to delete it or c to change it
- u to undo, <C-r> to redo
- y to copy / “yank” (some other commands like d also copy)
- p to paste
- Lots more to learn: e.g. ~ flips the case of a character

### Modifiers
- ci( change the contents inside the current pair of parentheses
- ci[ change the contents inside the current pair of square brackets
- da' delete a single-quoted string, including the surrounding single quotes

### Customize and Extend vim
- configure `~/.vimrc` e.g. [this custom config](https://missing.csail.mit.edu/2020/files/vimrc)
- extend vim by creating the directory `~/.vim/pack/vendor/start/` and put [plugins](https://vimawesome.com/) in there (e.g. git clone) 

## 4. Data Wrangling
### sed (stream editor) 
- `s` for subsitution
    - i.e. `s/REGEX/SUBSTITUTION/` to substitute on each line
    - e.g. `sed 's/.*.com Email Confirmed //'` to remove everything up until `aaa.com Email Confirmed`
- `i` for injecting text
- `p` for printing text... and more.
- `- E` flag for a more modern version of sed

### Regex Recap:
- . means “any single character” except newline
- * zero or more of the preceding match
- + one or more of the preceding match
- [abc] any one character of a, b, and c
- (RX1|RX2) either something that matches RX1 or RX2
- ^ the start of the line
- $ the end of the line
- `\w`, `\W`, `\d`, `\D`, `\s`

### Additional CLI commands:
- sort e.g. `sort -nk1,1`:
    - `sort -n` will sort in numeric (instead of lexicographic) order. 
    - `-k1,1` means “sort by only the first whitespace-separated column”. 
    - `,n` says “sort until the nth field, where the default is the end of the line. 

- head | tail -n#

### awk 
Another stream editor for processing text streams. For each row that is matched, process the entire row `$0`, or `$1` to `$n`th field separated (by default with whitespaces, -F to modify)
e.g. print the second field of each row where the first element is 1 and the second starts with h and ends in d. And then count the number of lines
` | awk '$1 == 1 && $2 ~ /^h[^ ]*d$/ { print $2 }' | wc -l`
Note: programming language e.g.
```
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

Also useful with mathematical operations
- ` | paste -sd+ | bc -l` (where bc is a precision calculator)
    "1 2 3" --> "1+2+3" --> (bc) --> 6
- `echo "2*($(data | paste -sd+))" | bc -l`

## 5. Command Line Environment
### Job Control
Your shell is using a UNIX communication mechanism called a signal to communicate information to the process. When a process receives a signal it stops its execution, deals with the signal and potentially changes the flow of execution based on the information that the signal delivered. For this reason, signals are software interrupts.
e.g. ^C sends a `SIGINT` to the process
We can also stop a process from being interrupted
```
#!/usr/bin/env python
import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1
```
Now you would need to send a quit signal ^/

NOTE: `SIGINT` and `SIGQUIT` are usually associated with terminal related requests but a more generic signal for asking a process to exit gracefully is the `SIGTERM` signal. To send this signal we can use the kill command, with the syntax kill -TERM <PID>.
```
     1       HUP (hang up)
     2       INT (interrupt)
     3       QUIT (quit)
     6       ABRT (abort)
     9       KILL (non-catchable, non-ignorable kill)
     14      ALRM (alarm clock)
     15      TERM (software termination signal)
```
### Pausing and backgrounding processes
`SIGSTOP` ^Z - pauses a process, which can then be continued to the foreground or in the background using `fg` or `bg`, respectively.

- `jobs`: shows status of all jobs
- `pgrep job_name`: returns PIDs of any running processes with a matching command string

Note: all processes created are associated with your terminal and will die when the terminal is closed i.e. process receives a `SIGHUP` signal.<br>
To prevent that from happening you can run the program with `nohup` (a wrapper to ignore `SIGHUP`), or use `disown` if the process has already been started.

Finally, `SIGKILL` (which cannot be captured by the process) will always terminate it immediately. However, it can have bad side effects such as leaving orphaned children processes.

### Shortcuts
- Ctrl + a: Go to the beginning of the line
- Ctrl + e: Go to the end of the line
- Ctrl + XX: Go to the beginning of the line, and pressing Ctrl + XX again takes you back where you were.
- Ctrl + u: Delete current line
- Ctrl + _: Undo previous key stroke
- Alt + f / Escape + F: Move forward, one-word
- Alt + b / Escape + B: Move backwards, one-word
- Alt + d: Delete all characters after the cursor
- Alt + u: Up-case all characters after the cursor
- Alt + t: Swap current word with the previous word

### Terminal Multiplexers
Terminal multiplexers let you detach a current terminal session and reattach at some point later in time. Ideal for remote machines since it voids the need to use `nohup` and similar tricks

[tmux](http://man7.org/linux/man-pages/man1/tmux.1.html)
- keybindings form <C-b> x where that means press Ctrl+b release, and the press x. tmux has the following hierarchy of objects
- <b>Sessions</b>: independent workspace with one or more windows
    - 'tmux' starts a new session.
    - 'tmux new -s NAME' starts it with that name.
    - 'tmux ls' lists the current sessions
    - Within `tmux` typing `<C-b> d` dettaches the current session
    - `tmux a` attaches the last session. You can use `-t` flag to specify which
- <b>Windows</b>: similar to tabs, they are visually separate parts of the same session
    - `<C-b> c` Creates a new window. To close it you can just terminate the shells doing <C-d>
    - `<C-b> N` Go to the N th window. Note they are numbered
    - `<C-b> p` Goes to the previous window
    - `<C-b> n` Goes to the next window
    - `<C-b> ,` Rename the current window
    - `<C-b> w` List current windows
- <b>Panes</b>: multiple shells in the same visual display
    - `<C-b> "` Split the current pane horizontally
    - `<C-b> %` Split the current pane vertically
    - `<C-b> <direction>` Move to the pane in the specified direction. Direction here means arrow keys.
    - `<C-b> z` Toggle zoom for the current pane
    - `<C-b> [` Start scrollback. You can then press `<space>` to start a selection and `<enter>` to copy that selection.
    - `<C-b> <space>` Cycle through pane arrangements.

[tmux tutorial](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)
- status bar: opened windows (left) & system information (right).
- Split Panes: `<C-b>` then `%` (horizontally) or `"` (vertically)
- Create Windows: `<C-b>` then `c` (create), `p` (for previous window) and `n` (for next) or `#` (for the specific number of that window)
- Detach Session: `<C-b>` then `d` or `D` (to select a session), which will leave it running in the background.
- `tmux ls` and `tmux attach -t 0`
- `tmux new -s database`
- `tmux rename-session -t 0 database` and then `tmux attach -t database`
- `C-b z`: make a pane go full screen. Hit C-b z again to shrink it back to its previous size
- `C-b C-<arrow key>`: Resize pane in direction of <arrow key>4
- `C-b ,`: Rename the current window


### Aliases
### Dotfiles and Portability
### Remote Machines 
- Executing Commands
- SSH Keys and Key Generation
- Copy files over
### Port Forwarding
Credit to [this stackoverflow post](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)
<b>Local Port Forwarding</b><br>
e.g. software on remote server listening to port 5000 and we want to forward it to our localhost port 8888. `ssh -L 5000:localhost:8888 foobar@remote_server`<br>
![local](https://i.stack.imgur.com/a28N8.png)
local: -L Specifies that the given port on the local (client) host is to be forwarded to the given host and port on the remote side.

ssh -L sourcePort:forwardToHost:onPort connectToHost means: connect with ssh to connectToHost, and forward all connection attempts to the local sourcePort to port onPort on the machine called forwardToHost, which can be reached from the connectToHost machine.<br><br>
<b>Remote Port Forwarding</b>
![remote](https://i.stack.imgur.com/4iK3b.png)
remote: -R Specifies that the given port on the remote (server) host is to be forwarded to the given host and port on the local side.

ssh -R sourcePort:forwardToHost:onPort connectToHost means: connect with ssh to connectToHost, and forward all connection attempts to the remote sourcePort to port onPort on the machine called forwardToHost, which can be reached from your local machine.

### SSH Configurations
`./ssh/config`
```
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    RemoteForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz
```
Server side configuration is usually specified in /etc/ssh/sshd_config. Here you can make changes like disabling password authentication, changing ssh ports, enabling X11 forwarding, &c. You can specify config settings in a per user basis

## 6. Version Control (Git)
. <root> (tree)
├── foo (tree)
│   └── bar.txt (blob, contents = "hello world")
├── baz.txt (blob, contents = "git is wonderful")

```
// a file is a bunch of bytes
type blob = array<byte>

// a directory contains named files and directories
type tree = map<string, tree | file>

// a commit has parents, metadata, and the top-level tree
type commit = struct {
    parent: array<commit>
    author: string
    message: string
    snapshot: tree
}
```

NOTE: Commits are immutable i.e. “edits” to the commit history are actually creating entirely new commits, and references are updated to point to the new ones.

### Objects and content-addressing
`type object = blob | tree | commit`

All objects are content-addressed by their [SHA-1 hash](https://en.wikipedia.org/wiki/SHA-1) i.e. 40 hexadecimal characters.
```
objects = map<string, object>

def store(object):
    id = sha1(object)
    objects[id] = object

def load(id):
    return objects[id]
```

Blobs, trees, and commits are unified in this way: they are all objects. When they reference other objects, they contain the other objects' reference.

e.g. observe a tree reference `git cat-file -p 698281bc680d1995c5f4caaf3359721a5a58d48d` which contains a blob (baz.txt) and a dir (foo)
```
100644 blob 4448adbf7ecd394f42ae135bbeed9676e894af85    baz.txt
040000 tree c68d233a33c5c06e0340e4c224f0afca87c8ce87    foo
```
Then observe the contents of the blob with the hash `git cat-file -p 4448adbf7ecd394f42ae135bbeed9676e894af85`
```
git is wonderful
```

### References
All snapshots can be identified by their SHA-1 hash. That’s inconvenient, because humans aren’t good at remembering strings of 40 hexadecimal characters. Git’s solution to this problem is human-readable names for SHA-1 hashes, called `references`. 

References are `pointers` to commits. Unlike objects, which are immutable, references are `mutable` (can be updated to point to a new commit). For example, the master reference usually points to the latest commit in the main branch of development.

```
references = map<string, string>

def update_reference(name, id):
    references[name] = id

def read_reference(name):
    return references[name]

def load_reference(name_or_id):
    if name_or_id in references:
        return load(references[name_or_id])
    else:
        return load(name_or_id)
```

One detail is that we often want a notion of “where we currently are” in the history, so that when we take a new snapshot, we know what it is relative to. This is a special reference called “HEAD”.

### Advanced Git
- `git config`: Git is highly customizable
- `git clone --shallow`: clone without entire version history
- `git add -p`: interactive staging
- `git rebase -i`: interactive rebasing
- `git blame`: show who last edited which line
- `git stash`: temporarily remove modifications to working directory
- `git bisect`: binary search history (e.g. for regressions)
- `.gitignore`: specify intentionally untracked files to ignore
- `git log --all --graph --decorate --online`: visual display of history (alias to `git-graph`)
- `git-bisect`: binary search to find the commit that introduced a bug

## 7. Debugging and Profiling
- Printing vs Logging: Logging can have different levels
- many systems produce logs --> typically in `/var/log` e.g. NGINX logs found in `var/log/nginx`

### system log
Recently, systems have started using a system log, which is increasingly where all of your log messages go. `systemd`: a system daemon that controls many things in your system such as which services are enabled and running. 
- `systemd` places the logs under `/var/log/journal` in a specialized format
- UNIX: use the `journalctl` command to display the messages. 
- On MacOS: `log show --last 10s`

```
logger "Hello Logs"
# On macOS
log show --last 1m | grep Hello
# On Linux
journalctl --since "1m ago" | grep Hello
```

### Debugger
- often a wrapper for your function e.g. `pdb` (python debugger), `ipdb` (improved debugger for IPython), `gdb`

ipdb commands:
- s: step
- c: continue until crash
- p var: print variable
- l: list code
- b #: breakpoint at line #
- p locals(): print local variables

Static Analysis Tools (preliminary check if function will fail)
- `pyflakes foo.py`
- `mypy foo.py`
- `writegood bar.txt`

### (CPU) Profiling (Tracing vs Sampling)
- tracing profilers follow the code & note the time at each step (large overhead)
    - `cProfile`
    - `kernprof`
    - `memory-profiler`
- sampling profilers halt the program every X seconds, review stack and take not of time. Therefore more accurate.

#### A note on Time
- `Real`: Wall clock elapsed time from start to finish of the program, including the time taken by other processes and time taken while blocked (e.g. waiting for I/O or network)
- `User`: Amount of time spent in the CPU running user code
- `Sys`: Amount of time spent in the CPU running kernel code

e.g.
```
$ time curl https://missing.csail.mit.edu &> /dev/null`
real    0m2.561s
user    0m0.015s
sys     0m0.012s
```

[Flame Graph](http://www.brendangregg.com/flamegraphs.html)<br>
![flame](https://camo.githubusercontent.com/a09235f122f62dbd0cedca7219b28a76a6f1d387/687474703a2f2f6474726163652e6f72672f626c6f67732f6272656e64616e2f66696c65732f323031312f31322f6d7973716c2d666c616d652d3630302e706e67)<br>

[Call Graph](https://blackfire.io/docs/book/09-call-graphs-galore)<br>
![call](https://blackfire.io/docs/callgraph-simple.png)

### Resource Monitoring
- General Monitoring: `htop`, `dstat`
- I/O operations: `iotop`
- Disk Usage: `df` (metrics per partitions), `du` (disk usage per file for the current directory) and `ncdu` (interactive to navigate and delete files). `-h` for human readable format.
- Memory Usage: `htop`, `free` (total amount of free and used memory in the system)
- Open Files: `lsof` (lists file information about files opened by processes)
- Network Connections and Config: `ss` (monitor incoming and outgoing network packets statistics). A common use case of ss is figuring out what process is using a given port in a machine. For displaying routing, network devices and interfaces you can use `ip`. Note that `netstat` and `ifconfig` have been deprecated in favor of the former tools respectively.
- Network Usage: `nethogs` & `iftop` (good interactive CLI tools for monitoring network usage)

## 8. Metaprogramming
_Processes surrouding the work involved in programming_ i.e. how systems are built, tested and handle dependencies

### Build Systems
Encoding a list of commands to build your program into a tool e.g. `make`
- Target e.g. a file (paper.pdf) or a list of tests
- Dependencies e.g. list of things to be built/installed prior to build
- Rules: list of commands e.g. how to go from dependencies to a target

```
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

- LHS is the `target` and RHS of : are the `dependencies`
- Below the target and dependencies are the `list of rules` 
- `%` is a wildcard string
- `$*` matches to the provided wildcard associated with `%`
- `$@` matches to name of target

NOTE: `make` will only rebuild if any changes occurred to dependencies.

### Dependency Management
Dependencies are available through a repository that hosts a large number of such dependencies in a single place, and provides a convenient mechanism for installing them. 
- `apt`: Ubuntu package repositories for Ubuntu system packages
- `RubyGems`: Ruby libraries
- `PyPi`: Python libraries
- `Arch User Repository`: Arch Linux user-contributed packages.

### Versioning
[Semantic Versioning](https://semver.org/): `major.minor.patch`
- patch version: new release doesn't change the API
- minor version: add/remove/rename to your API in a backwards-compatible way.
- major version: change the API in a non-backwards-compatible way

#### lock file
A file that lists the exact version you are currently depending on of each dependency. 
- avoiding unnecessary recompiles
- reproducible builds
- not automatically updating to the latest version (which may be broken). 

#### vendoring
Extreme version of dependency locking where you copy all the code of your dependencies into your own project. 
- total control over any changes to it
- can introduce your own changes to it
- but have to explicitly pull in any updates from the upstream maintainers.

### Continuous integration systems
For additional tasks required when changes are made e.g. 
- upload a new version of the documentation
- upload a compiled version somewhere
- release the code to pypi
- run your test suite,

<b>Broad CI Platforms</b> e.g. Travis CI, Github Actions etc.<br>
<b>Narrow CI Platforms</b> e.g. CodeCov, Dependabot etc.

By far the most common one is a rule like “when someone pushes code, run the test suite”. When the event triggers, the CI provider spins up a virtual machines (or more), runs the commands in your `recipe`, and then usually notes down the results somewhere. You might set it up so that you are notified if the test suite stops passing, or so that a little badge appears on your repository as long as the tests pass.

### Testing
- `Test suite`: a collective term for all the tests
- `Unit test`: a “micro-test” that tests a specific feature in isolation
- `Integration test`: a “macro-test” that runs a larger part of the system to check that different feature or components work together.
- `Regression test`: a test that implements a particular pattern that previously caused a bug to ensure that the bug does not resurface.
- `Mocking`: the replace a function, module, or type with a fake implementation to avoid testing unrelated functionality. For example, you might “mock the network” or “mock the disk”.

## 9. Security & Cryptography
### Entropy
- measure of randomness: useful when determining the strength of a password.
- `entropy = log_2(# of possibilities)`: measured in bits

### Hash Functions
A [cryptographic hash function](https://en.wikipedia.org/wiki/Cryptographic_hash_function) maps data of arbitrary size to a fixed size, and has some special properties. A rough specification of a hash function is as follows
```
hash(value: array<byte>) -> vector<byte, N>  (for some fixed N)
```

e.g. [SHA1](https://en.wikipedia.org/wiki/SHA-1) hash function (used in git) maps an arbitrary sized input to 160-bit outputs (40 hexadecimal characters)

#### Hash Properties
- `Deterministic`: same input generates same output.
- `Non-invertible`: hard to find an input `m` such that `hash(m) = h` for some desired output `h`.
- `Target collision resistant`: given an input `m_1`, it’s hard to find a different input `m_2` such that `hash(m_1) = hash(m_2)`.
- `Collision resistant`: it’s hard to find two inputs `m_1` and `m_2` such that `hash(m_1) = hash(m_2)` (note that this is a strictly stronger property than target collision resistance).

NOTE: while it may work for certain purposes, SHA-1 is [no longer](https://shattered.io/) considered a strong cryptographic hash function.

#### Applications
- Git, for content-addressed storage.
- A short summary of the contents of a file. When downloading mirrored software, official sites will post the hashes alongside download links to compare against mirrored links.
- Commitment schemes e.g. I want to commit to a particular value but only reveal that value later. I select a value `r` and share `h` (which comes from `h = sha256(r)` and afterwards I can demonstrate that in fact `h = sha256(r)`, thus confirming I initially committed to `r`.

### Key Derivation Functions (KDF)
Similar to hash functions but deliberately slow to compute in order to prevent brute-force attacks e.g. `PBKDF2`

#### Applications
- Use in tandem with other applications i.e. producing keys from passphrases for use in other cryptographic algorithms (e.g. symmetric cryptography, see below).
- Storing login credentials. Do not store plaintext passwords; rather generate and store a random [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) `salt = random()` for each user, store KDF(password + salt), and verify login attempts by re-computing the KDF given the entered password and the stored salt.

### Symmetric Cryptography
- keygen() -> key  (this function is randomized)
- encrypt(plaintext: array<byte>, key) -> array<byte>  (the ciphertext)
- decrypt(ciphertext: array<byte>, key) -> array<byte>  (the plaintext)

#### Properties
- Noninvertible ciphertext without key
- `decrypt(encrypt(m, k), k) = m`

#### Applications
- Encrypting files for storage in an untrusted cloud service. This can be combined with KDFs, so you can encrypt a file with a passphrase. Generate key = KDF(passphrase), and then store encrypt(file, key).

passphrase      -->     KDF     -->     key
                                        ├── encrypt --> ciphertext
`(don't need to remember the key)`      plain text

### Asymmetric Cryptography
Two keys: a private key and a public key.
- keygen() -> (public key, private key)  (this function is randomized)
- encrypt(plaintext: array<byte>, public key) -> array<byte>  (the ciphertext)
- decrypt(ciphertext: array<byte>, private key) -> array<byte>  (the plaintext) i.e. `decrypt(encrypt(m, public key), private key) = m.`
- sign(message: array<byte>, private key) -> array<byte>  (the signature)
- verify(message: array<byte>, signature: array<byte>, public key) -> bool  (whether or not the signature is valid) e.g. `verify(message, sign(message, private key), public key) = true`

#### Applications
- PGP email encryption. post a public key online (e.g. in a PGP keyserver, or on Keybase) and anyone can send you an encrypted email
- Private messaging. Apps like Signal and Keybase use asymmetric keys to establish private communication channels.
- Signing software. Git can have GPG-signed commits and tags. With a posted public key, anyone can verify the authenticity of downloaded software.

#### Key Distribution
- Challenges arise when distributing public keys / mapping public keys to real-world identities. 
- Many solutions to this problem. 
    - Signal has one simple solution: trust on first use, and support out-of-band public key exchange (you verify your friends’ “safety numbers” in person). 
    - PGP has a different solution, which is web of trust. 
    - Keybase has yet another solution of social proof (along with other neat ideas).

#### Case Studies
- Password Managers: Various tools exist (e.g. [KeePassXC](https://keepassxc.org/)) that use unique, randomly generated high-entropy passwords and save them in one place, encrypted with a symmetric cipher with a key produced from a passphrase using a KDF. 
- Two-factor authentication (2FA)
- Full disk encryption: encrypts the entire disk with a symmetric cipher, with a key protected by a passphrase
- Private Messaging: End-to-end security is bootstrapped from asymmetric-key encryption. Obtaining your contacts’ public keys is the critical step here. If you want good security, you need to authenticate public keys out-of-band (with Signal or Keybase), or trust social proofs (with Keybase).
- SSH: `ssh-keygen` generates an asymmetric keypair
    - `ssh-keygen` program prompts user for a passphrase, which is fed through a key derivation function to produce a key, which is then used to encrypt the private key with a symmetric cipher.
    - once the server knows the client’s public key (stored in the `.ssh/authorized_keys` file), a connecting client can prove its identity using asymmetric signatures using [challenge-response](https://en.wikipedia.org/wiki/Challenge%E2%80%93response_authentication) 
    - server picks a random number and send it to the client
    - client then signs this message and sends the signature back to the server, which checks the signature against the public key on record
    - this proves that the client is in possession of the private key corresponding to the public key that’s in the server’s `.ssh/authorized_keys` file, so the server can allow the client to log in

## 10. Random Topics
### Daemons
- Computer program that runs as a background process. Programs often ending in a `d` e.g. 
- `sshd`: responsible for listening to incoming SSH requests and confirming whether the remote user has credentials to log in.
- `systemd`: contains a list of system background processes
- `systemctl status`: list the current running daemons

### Filesystems in User Space (FUSE)
- OS supports different file systems within the kernel interface but you can create your own filesystem
- FUSE allows you to write your own commands and it will bridge necessary calls to kernel interface
- `sshfs`: open remote files through SSH connections
- `rclone`: mount cloud storage services (Dropbox, GDrive, S3 etc) and navigate locally
- `gocryptfs`: encrypt/decrypt files on the fly when navigating

### Hammerspoon (Automation for MacOS)
- write LUA scrips that hook into OS Functionality e.g.
    - bind hotkeys to move windows around
    - create menu bar button that autommatically lays out windows in a specific arrangement i.e. for home setup, lab setup etc.
    - mute speakers when you arrive somewhere (WIFI network detection)
    - show warning if a different power supply is connected

### Where are programs located?
[Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)
- `/bin`: essential command binaries e.g. `cat`, `ls`, `cp`
- `/sbin`:  essential system binaries
- `/dev`: device files i.e. interfaces to hardware devices e.g. `/dev/null`, `/dev/disk0`
- `/etc`: host-specific system-wide config files
- `/home`: home directory for users in the system
- `/lib`: common libraries for system programs
- `/opt`: optional (3rd party) application software
- `/sys`: information and configuarions for the system
- `/tmp`: temporary files (also /var/tmp) usually deleted between reboots
- `/usr`: read-only user data
    - /usr/bin: non-essesstion bin files
    - /usr/sbin: nonessential sbin files
    - /usr/local/bin: user-compiled binary programs
- `/var`: variable files e.g. logs, caches, lock files