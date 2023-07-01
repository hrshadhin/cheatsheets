# tmux cheatsheet

## Index
- [Just start](#install-tmux)
- [List all shortcuts](#list-all-shortcuts)
- [Sessions](#sessions)
- [Windows (tabs)](#windows-tabs)
- [Panes (splits)](#panes-splits)
- [Sync Panes](#sync-panes)
- [Resizing Panes](#resizing-panes)
- [Copy Mode](#copy-mode)
- [Misc](#misc)
- [Advance search](#advance-search)
- [Configurations](#configurations-options)

## Install *tmux*:
```bash
# Debian
sudo apt install tmux
# Arch
sudo pacman -S tmux
# Mac OSX
brew install tmux
```

start new:
```
tmux
```

start new with session name:
```
tmux new -s myname
```

attach:
```
tmux a  #  (or at, or attach)
```

attach to named:
```
tmux a -t myname
```

list sessions:
```
tmux ls
```

kill session:
```
tmux kill-session -t myname
```

Kill all the tmux sessions:
```
tmux ls | grep : | cut -d. -f1 | awk '{print substr($1, 0, length($1)-1)}' | xargs kill
```

Shell alias:
```bash
# Attaches tmux to a session (example: ta dev)
alias ta='tmux attach -t'
# Creates a new session
alias tn='tmux new-session -s '
# Kill session
alias tk='tmux kill-session -t '
# Lists all ongoing sessions
alias tl='tmux list-sessions'
# Detach from session
alias td='tmux detach'
# Tmux Clear pane
alias tc='clear; tmux clear-history; clear'

```

In tmux, hit the prefix `ctrl+b` (my modified prefix is `ctrl+a`) and then:

>  *CKB = Custom Keybindings

## List all shortcuts
to see all the shortcuts keys in tmux simply use the `bind-key ?` in my case that would be `CTRL-B ?`

## Sessions
```
:new<CR>  new session
s  list sessions
$  name session
```

## Windows (tabs)
```
c  create window
w  list windows
n  next window
p  previous window
f  find window
,  name window
&  kill window
```

## Panes (splits) 
```
% or | vertical split *CKB
" or - horizontal split *CKB

o  swap panes
q  show pane numbers
x  kill pane
<prefix> q (Show pane numbers, when the numbers show up type the key to goto that pane)
<prefix> { (Move the current pane left)
<prefix> } (Move the current pane right)
<prefix> hjkl (Move the current pane left/down/up/right) *CKB
<prefix> z toggle pane zoom
```

## Sync Panes 

You can do this by switching to the appropriate window, typing your Tmux prefix (commonly Ctrl-B or Ctrl-A) and then a colon to bring up a Tmux command line, and typing:

```
:setw synchronize-panes
```

You can optionally add on or off to specify which state you want; otherwise the option is simply toggled. This option is specific to one window, so it won’t change the way your other sessions or windows operate.

## Resizing Panes

You can also resize panes if you don’t like the layout defaults. I personally rarely need to do this, though it’s handy to know how. Here is the basic syntax to resize panes:
```
PREFIX : resize-pane -D (Resizes the current pane down)
PREFIX : resize-pane -U (Resizes the current pane upward)
PREFIX : resize-pane -L (Resizes the current pane left)
PREFIX : resize-pane -R (Resizes the current pane right)
PREFIX : resize-pane -D 20 (Resizes the current pane down by 20 cells)
PREFIX : resize-pane -U 20 (Resizes the current pane upward by 20 cells)
PREFIX : resize-pane -L 20 (Resizes the current pane left by 20 cells)
PREFIX : resize-pane -R 20 (Resizes the current pane right by 20 cells)
PREFIX : resize-pane -t 2 20 (Resizes the pane with the id of 2 down by 20 cells)
PREFIX : resize-pane -t -L 20 (Resizes the pane with the id of 2 left by 20 cells)
PREFIX HJKL (Resize the pane left/down/up/right by 5 cells) *CKB
```
    
## Copy Mode:

Pressing `PREFIX [` places us in Copy mode. We can then use our movement keys to move our cursor around the screen. By default, the arrow keys work. we set our configuration file to use Vim keys for moving between windows and resizing panes so we wouldn’t have to take our hands off the home row. tmux has a vi mode for working with the buffer as well. To enable it, add this line to `.tmux.conf`:

    setw -g mode-keys vi

With this option set, we can use h, j, k, and l to move around our buffer.

To get out of Copy mode, we just press the `ENTER` key. Moving around one character at a time isn’t very efficient. Since we enabled vi mode, we can also use some other visible shortcuts to move around the buffer.

For example, we can use `"w"` to jump to the next word and `"b"` to jump back one word. And we can use `"f"`, followed by any character, to jump to that character on the same line, and `"F"` to jump backwards on the line.
```
Function                vi             emacs
Back to indentation     ^              M-m
Clear selection         Escape         C-g
Copy selection          Enter          M-w
Cursor down             j              Down
Cursor left             h              Left
Cursor right            l              Right
Cursor to bottom line   L
Cursor to middle line   M              M-r
Cursor to top line      H              M-R
Cursor up               k              Up
Delete entire line      d              C-u
Delete to end of line   D              C-k
End of line             $              C-e
Goto line               :              g
Half page down          C-d            M-Down
Half page up            C-u            M-Up
Next page               C-f            Page down
Next word               w              M-f
Paste buffer            p              C-y
Previous page           C-b            Page up
Previous word           b              M-b
Quit mode               q              Escape
Scroll down             C-Down or J    C-Down
Scroll up               C-Up or K      C-Up
Search again            n              n
Search backward         ?              C-r
Search forward          /              C-s
Start of line           0              C-a
Start selection         Space          C-Space
Transpose chars                        C-t
Paste buffer            P *CKB             
```
`Start selection` in vim mode via `"v"` & copy with `"y"`

## Misc
```
d  detach
t  big clock
?  list shortcuts
:  prompt
= Choose which buffer to paste interactively from a list
] Paste the most recently copied buffer of text
```

## Advance search
`PREFIX + [ => ? + KEYWORD` or `PREFIX + [ => / + KEYWORD`
```
? + [a-z]+_1
```
Regex breakdown:
- `[a-z]` is the lowercase a-z
- `+` means one or more subsequent characters (which was a-z). `[a-z]+` means one or more any lowercase alphabet character
- `_1` is a literal underscore followed by a literal one.

This will match `redis_1`, `node_1`, and `rails_1` (it will also match strings like `java_1`, `sidekiq_1`, etc.

If you want to match ONLY `redis_1`, `node_1`, or `rails_1` you can use a group match:
```
? + (redis|node|rails)_
```

## Configurations Options:
```bash
# Set prefix to Ctrl-a instead of Ctrl-b
unbind C-b
set -g prefix C-a
bind a send-prefix

# Split windows using | and -
unbind '"'
unbind %
bind | split-window -h
bind - split-window -v

# Reload
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# Resize panes HJKL
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# Traverse panes hjkl
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# General settings
set -g default-terminal "screen-256color"
set -g @continuum-restore 'on'
set -g status-interval 10
set -s escape-time 0
set -g base-index 1
setw -g pane-base-index 1
set -g status-justify centre

# Activity monitoring
setw -g monitor-activity on
set -g visual-activity on

# Vim key bindings
setw -g mode-keys vi
bind -T copy-mode-vi v send -X begin-selection
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "pbcopy"
bind P paste-buffer
bind -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel "pbcopy"

# For *nix system remove "pbcopy"
```

