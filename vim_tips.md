# Vim tips
## Notes
In all code examples * denotes the cursor position.
_re_ after a command and in front of another command means that the default command which is the first one is remapped to the second, e.g. `<C-]>` _re_`<C-j>`.

## Normal mode
### Shortcuts

| Longhand | Compound | Descriptions                                               |
| -------- | -------- | ------------                                               |
| `cl`     | `s`      | Replaces the underlying character but stays in insert mode |
| `^C`     | `S`      | Changes the whole line independent of cursor position      |
| `U`      |          | Undo all changes in the current line                       |
| `vim.slp.buf.hover()`  | `K`          | Shows documentation of symbol under cursor. Jumps to help section when already in help.  |
|          | `KK`          | Same as above, but it jumps into the floating window  |
| `vim.lsp.buf.definition()` | `<C-]>` _re_ `gd` | Go to definition                                        |
| `vim.lsp.buf.references()`    | `grr` | List all references to the symbol under the cursor in quifix list           |
| `vim.lsp.buf.rename()` | `grn` | Rename variable or function                                |
| `vim.lsp.buf.code_action()` | `gra` | Runs a code action                                         |
| `vim.lsp.buf.implementation()` | `gri` | Lists all implementations for the sysmbol under the cursor in the quickfix window   |
| `vim.lsp.buf.document_symbol()` | `gO` | Lists all symbols in the current buffer in the quickfix window  |
| `vim.lsp.buf.signature_help()` | `<C-s>` | Shows signature information of the symbol in insert mode |

### Copy & Paste

Vim's copy command is called yank `y`, paste is `p` and cut is `d`. They
use the unnamed register `""` if not specified otherwise. A specific
register can be used by preceding the command with `"{register}"`.
A register name can be a character from `a` to `z`. If a register is
addressed with an uppercase letter the content is appended to it. On the
other hand lowercase letters overwrite the content of the register.
Another special register is the black hole register `"_` which doesn't
store anything. This could also be a used in the example below.

A yank does not only put the content into the unnamed register but also
in the yank register `"0`. This is helpful when replacing something
with the last yanked content, because delete overwrites the unnamed
register. Example:

```
v*ar = 1; process(1);
yiw
2f1
var = 1; process(*1);
diw
var = 1; process(*);
"0P
var = 1; process(var);
```

Pasting with `p` places the yanked content after the cursor and `P`
before the cursor. Depending on the yanked content, before and after
are relative to either characters or lines if a whole line was yanked.
`gp` and `gP` have the same pasting behavior as the two previously
described commands, but they place the cursor at the end of the placed
text.

When the paste command is issued in visual mode, the selected text
is replaced. This is another possibility to solve the example above
without the yank register.

Pasting can also be achieved in insert mode by using `<C-r>{register}`.
The unnamed register can be addressed with `"`.

Using the systems paste command can result in strange formatted code
because Vim treats the in put as if someone was typing it and applies
indentation rules. To disable autoindent, abbreviations and other
features which are not useful when pasting, the `paste` option can be
set before pasting. The option can be unset with `:set nopaste`. It can
also be bound to a key like F5 with `:set pasttoggle=<f5>`. Pasting from
the clipboard register `"+` automatically enables the `paste` option.

`:reg "{register}` shows the content of the register or all registers
if no register is given.

Some more special registers:

| Register | Content/Function                                |
| -------- | ----------------------------------------------- |
| `"+`     | Clipboard                                       |
| `"*`     | Primary (most recently selected text)           |
| `"=`     | Expression register. Can be used as calculator. |
| `"%`     | Name of current file                            |
| `"#`     | Name of alternate file                          |
| `".`     | Last inserted text                              |
| `":`     | Last ex command                                 |
| `"/`     | Last search pattern                             |

### Operator commands

| Trigger | Effect         |
| ------- | ------         |
| `g~`    | Swap case      |
| `gu`    | Make lowercase |
| `gU`    | Make uppercase |
| `>`     | Shift right    |
| `<`     | Shift left     |

### Motions
#### Basic motion commands
| Command     | Motion                                     |
| -------     | ------                                     |
| `j`         | Down one real line                         |
| `gj`        | Down one display line                      |
| `g{motion}` | Motion on display lines                    |
| `k`         | Up one real line                           |
| `h`         | One character left                         |
| `l`         | One character right                        |
| `0`         | First character of line                    |
| `^`         | First nonblank character of line           |
| `$`         | End of line                                |
| `w`         | Forward to start of next word              |
| `b`         | Backward to start of current/previous word |
| `e`         | Forward to end of current/next word        |
| `ge`        | Backward to end of previous word           |

`w`, `b`, `e` and `ge` use the definition of a word as a sequence of
letters, digits and underscores or as a sequence of other nonblank
characters separated with whitespace. These commands have an uppercase
equivalent (`W`, `B`, `E`, `gE`) which uses the WORD definition. A WORD
is a sequence of nonblank characters separated with whitespace.

#### Search
Vim has character search commands, which are handy to navigate within a
line, because they move to the next occurrence of the specified
character:

| Command        | Description                                           |
| -------        | -----------                                           |
| `f{character}` | Forward to the next occurrence                        |
| `F{character}` | Backward to the next occurrence                       |
| `t{character}` | Forward to the character before the next occurrence   |
| `T{character}` | Backward to the character before the next occurrence  |
| `;`            | Repeat the last character search                      |
| `,`            | Repeat the last character search in reverse direction |

The search command is exclusive so you can use it in `d{motion}`
commands and search for the first word which should not be deleted:

	*This is an example
	d/an<CR>
	*an example

#### Text objects
Text objects can be used to select sections of text with particular
start and end patterns like code blocks `{}`, arrays `[]` or tags
`<a>...</a>`. To select the inside of such a section without the start
and end patterns use `i`, for the whole section use `a`.
After one of those two characters a second character indicates the kind
of section which should be selected:

| Character  | Selection   |
| ---------  | ---------   |
| `)` or `b` | `(...)`     |
| `}` or `B` | `{...}`     |
| `]`        | `[...]`     |
| `>`        | `<...>`     |
| `'`        | `'...'`     |
| `"`        | `"..."`     |
| `` ` ``    | `` `...` `` |
| `t`        | XML tag     |
| `w`        | word        |
| `W`        | WORD        |
| `s`        | sentence    |
| `p`        | paragraph   |

For word, WORD and sentence `a` includes one space after (or before) the word or
sentence, for the paragraph a blank line is included. Those commands are
not really motions, but can be used in visual and operator pending
mode.

#### Surround.vim

Surround.vim is plugin which can be used to surround a selection with
delimiters, change delimiters or delete delimiters. Delimiters can be
any kind of parenthesis or `"` and `'`, like text-objects.

| Old text         | Command | New text         | Description              |
| -------          | ------- | --------         | -----------              |
| `Hello w*orld`   | `ysiw)` | `Hello (world)`  | Add surroundings `ys`    |
| `"Hello *world"` | `cs"'`  | `'*Hello world'` | Change surroundings `cs` |
| `"Hello *world"` | `ds"`   | `*Hello world`   | Delete surroundings `ds` |

#### Jump between matching parenthesis

= `%` jumps between matching parenthesis like `()`, `{}` and `[]`, but it =
works also for XML tags and language constructs like `do end` and `if
end` in ruby when the _matchit_ plugin is enabled.

#### Jump list

`<C-o>` is like back in a browser and `<C-i>` like forward. `:jumps`
shows the jump list. Jumps are generally larger as single character
or word motions and can move between files. Some jump commands
are listed below:

| Command                       | Jumps to                                        |
| -------                       | ------                                          |
| `[line]G`                     | specified line                                  |
| /{pattern}                    | pattern                                         |
| `%`                           | matching parenthesis                            |
| `(`/`)`                       | start of previous/next sentence                 |
| `{`/`}`                       | start of previous/next paragraph                |
| `H`/`M`/`L`                   | start of top/middle/bottom of screen            |
| `gf`                          | file name under the cursor (suffixesadd option) |
| `<>` _re_`<Leader>g`       | definition of keyword under cursor            |
| `` `{mark}`` _re_ `<Leader>m` | a mark                                          |

Vim maintains multiple jump lists. Each is scoped to a window.

#### Change list

This is a record of made changes and can be queried with `:changes`.

| Command | Effect                                               |
| ------- | ------                                               |
| `g;`    | Go backward in change list                           |
| `g,`    | Go forward in change list                            |
| `gi`    | Go to last insert location and switch to insert mode |

#### Markers

To mark a location use `m{a-zA-Z}` the lowercase letters are local to
the file the uppercase letters are global. `´{mark}` move to the line of
the mark. `` `{mark}`` _re_`<C-m>{mark}` moves to the exact 
position. Vim defines some special markers:

| Shortcut | Description                                       |
| -------- | -----------                                       |
| `` `` `` | Position before the last jump within current file |
| `` `. `` | Location of last change                           |
| `` `^ `` | Location of last insertion                        |
| `` `[ `` | Start of last change or yank                      |
| `` `] `` | End of last change or yank                        |
| `` `< `` | Start of last visual selection                    |
| `` `> `` | End of last visual selection                      |

### Macros

A macro can be recorded by pressing `q{register}`. Recording is
indicated in the status line. By typing `q` again the recording is
stopped. The macro can be displayed with `:reg {register}`. `^[` is used
to indicate pressing of the escape key. A macro can be replayed with
`[count]@{register}`. The last macro can be repeated with `@@`. Playback is
aborted if one of the commands fails. This can be thought of as serial
execution. Parallel execution can be achieved with a visual selection
(using probably line-wise Visual mode) and then executing `'<,'>normal
@{register}`. This jumps over lines which produce an error (for example
if a search command finds nothing on the line). Commands can be added to
a macro if the uppercase letter is used for the register.

Macros can be edited directly in Vim as text by putting the content of
the register into the file. This can be achieved with `"{register}p` or
with `:put {register}`. The later always puts the content into a new
lines below the current line, which is preferable for editing a macro.
To get the edited text back into the register `"{register}dd` or (`:da`)
can be used. This could cause problems because it appends a trailing
`^J` for the line break. To avoid this, this longer version can be used:
`0"{register}y$dd`. An alternative to editing the macro in the text is
to use Vim script functions like `substitute()`. A list of functions can
be found in the help files: `:h function-list`. Substitute can be used
like: `:let @{register}=substitute(@{register}, '{pattern}',
'{replacement}', '{options like g}'`.

#### Executing macros in multiple files
Macros can be executed in multiple files by putting all files which
should be changed into the argument list (`:args {filepattern}`). The
macro can be recorded as usual. Since the macro will be applied to all
buffers, the current changes should be reverted with `:edit!`. `:argdo
normal @{register}` executes the macro in all files. An alternative
- which executes the macro in series - is to move to the next buffer with
`:next` (or `:wnext` which also saves the changes) at the end of the macro 
and execute it multiple times by prefixing the macro call with a count.

#### Example: Number items in a list

```
:let i = 1           // declare a Vim script variable i
qa                   // start macro recoding
I<C-r>=i<CR>) <ESC>  // go to insert mode, evaluate i and add ) 
:let i += 1          // increase i by 1
q                    // stop recording
jVG                  // go to next line and select until the end
:'<,'>normal @a      // execute macro for the selected lines
```

### Little things
#### Arithmetic
`<C-a>` adds `[count]` to the number at or after the cursor, `<C-x>`
subtracts. If no number is given the default is 1.

#### Help
`:h[elp] {topic}` is the built in help system. `:h index` for example shows
all key mappings.

## Insert mode

| Shortcut               | Description                             |
| --------               | -----------                             |
| `<C-h>`                | Same as backspace                       |
| `<C-w>`                | Delete back one word                    |
| `<C-u>`                | Delete back to start of line            |
| `<C-o>`                | Switch to Insert Normal mode            |
| `<C-r>{register}`      | Paste from register                     |
| `<C-r><C-p>{register}` | Paste from register and fix indentation |
| `<C-r>=`               | Evaluate expression (Calculator)        |
| `<C-T>`                | Insert one shiftwidth of indent         |
| `<C-D>`                | Delete one shiftwidth of indent         |
| `<C-v>{code}`          | Enter character by decimal code         |
| `<C-v>u{code}`         | Enter character by hexadecimal code     |
| `<C-v>{nondigit}`      | Enter nondigit literally                |
| `<C-k>{char1}{char2}`  | Enter character by digraph              |

Insert Normal mode can be used to trigger one normal mode command, after
which Vim returns automatically to insert mode. For example `<C-o>zz`
centers the screen vertically and can be executed during writing in
insert mode.

Some normal mode commands which goes along the above shown shortcuts

| Command     | Description                          |
| -------     | -----------                          |
| `ga`        | Show code for character under cursor |
| `:digraphs` | Show a table of digraphs             |

### Replace mode

Replace mode can be issued from Normal mode with `R`. It behaves like
other editor when the insert key was pressed. Tab characters can pose a
problem in Replace mode if the mode is started on one of the 'spaces'
representing the tab. An alternative is the Virtual Replace mode which
handles the tab character like some space characters and is triggered
with `gR`.


## Visual mode

Visual mode can be used to select text visually like in other editors
when selecting text with the mouse. It is triggered with `v` followed by
movement commands. In Visual mode the free end is indicated by the
blinking cursor. The free end can be switched between the start of the
selection and the end of the selection by pressing `o`.

`V` starts the line-wise Visual mode. This can for example be used for
line-wise visual command which can also be repeated: `Vj>.` selects the
current and next line and indents them twice.

In vernal operators should be preferred to Visual commands if possible,
because the later doesn't always work with the dot command in an
expected way because the command is repeated for the same selection
size. So a command on a word with `vit` might fail when repeated in the
next line on another word, if that word has a different length.

The `U` command in Visual mode changes the selection to upper case.

`<C-v>` enables the block wise Visual mode. This can be used to edit
text on more than one line. For example adding a ';' to three lines:
`<C-v>jj$A;<ESC>`. The changes in text are only displayed on the first
line and are processed for the other selected lines when the Visual mode
is left by pressing escape.

### Select mode

Select mode differs from Visual mode in that the selected text can be
replaced by starting to type. Switching between Visual and Select modes
can be achieved with `<C-g>`.

## Command-Line mode

This mode stems from the 'ancestors' of Vim: the line editors ex and ed.
Command-Line mode is started by typing `:` in Normal mode. Afterwards an
Ex command can be entered and executed with `<CR>`. This returns us to
Normal mode. Another way to exit Command-Line mode is to use `<Esc>`.
The basic structure of an Ex Command is: `:[range]{command}
[arguments]`. Since ex was a line editor many line based tasks can be
achieved with Ex commands (the optional range is omitted):

| Command                         | Explanation                                                   |
| -------                         | -----------                                                   |
| `:delete [x]`                   | Delete specified lines [into register x]                      |
| `:yank [x]`                     | Yank specified lines [into register x]                        |
| `:[line]put [x]`                | Put the text form register x after the specified line         |
| `:copy {address}` or `:t {...}` | Copy specified lines to below the line specified by {address} |
| `:m[ove] {address}`             | Move specified lines to below the line specified by {address} |
| `:join`                         | Join the specified lines                                      |
| `:normal {commands}`            | Execute Normal mode {commands} on each specified line         |
| `:s[ubstitute] ...`             | see [Substitution](#Substitution)                             |
| `:global /{pattern}/[cmd]`      | Execute Ex [cmd] on all specified lines matching {pattern}    |

If the range is not given, the current line is used. The range can be
specified with the following symbols:

| Symbol        | Meaning                                         |
| ------        | -------                                         |
| `{number}`    | Addresses the line number of the file           |
| `$`           | Addresses the last line of the file             |
| `0`           | Addresses the virtual line above the first line |
| `.`           | Addresses the line where the cursor is placed   |
| `'m`          | Addresses the line containing the mark m        |
| `'<`          | Addresses the start of the visual selection     |
| `'>`          | Addresses the end of the visual selection       |
| `%`           | Addresses the entire file                       |
| `/{pattern}/` | Addresses the line which matches the pattern    |

A range can be specified by combining two symbols separated by a comma,
e.g. `1,$` also addresses the entire file. Each address can be offset in
both directions: `:{address}+n` or `:{address}-n`

Insert mode commands like `<C-w>`, `<C-u>` and `<C-r>{register}` work
also in Command-Line mode.

The last Ex command is repeated with `@:`, because the : register
contains the last executed command. After once repeating the command
with `@:`, the command can be repeated with `@@`.

Typing of commands can be shortened by using autocompletion with
`<Tab>`. The history of executed Ex commands can be traversed with
`<Up>` and `<Down>` __re__ `<C-p>` and `<C-n>`.

### Global commands

As shown in the overview table, the global command has the following
form: `:[range] global[!] /{pattern}/ [cmd]`. In contrast to most other
Ex commands, the default range is the entire file (`%`) and not the
current line (`.`). The pattern is part of the search history and can be
left black for using the last search pattern. If the command is not
specified, Vim uses `:print` as default. The command can be executed by
all lines not matching the pattern when `:global!` or `vglobal` is used.

For example to delete everything but the content of HTML tags,
one could use the following command: `:g/\v\<\/?\w+>/d`. Another example
is to collect all TODO items in a register. First all TODO items in a
file could be displayed with: `:g/TODO`. Before using the register "a,
it is cleared with `qaq`. Now the TODO items can be added to the
register: `:g/TODO/yank A` and afterwards displayed using `:reg a`. This
approach scales well in combination with `:argdo` and `:bufdo` for
multiple files. A complex example is to sort the properties of each rule
in a CSS file. The command which can achieve this is: `:g/{/ .+1,/}/-1
sort`. The first part searches for the line where a rule starts (e.g.
'body {'). The range for the sort command is generated relative to each
line found by `:g/{/`, which can be addressed with `.`. It starts one
line below the start of the rule and goes until the line before the
closing of the rule '}'. The `:>` command can be used before using sort
to see how many lines are selected for each global match.

### Interaction with the shell

Vim can call shell commands and outputs from the shell commands can be
inserted as text.

| Command                | Description                                                    |
| -------                | -----------                                                    |
| `:shell`               | Starts a shell (return to Vim by typing exit)                  |
| `:!{cmd}`              | Execute {cmd} with the shell (return with enter)               |
| `:read !{cmd}`         | Execute {cmd} and insert its standard output below the cursor   |
| `:[range]write !{cmd}` | Execute {cmd} with [range] lines as standard input             |
| `:[range]!{filter}`    | Filter the specified [range] through external program {filter} |
| `!{motion}`            | Start Command-line mode prepopulated with corresponding range  |

## Files

The `:edit {filepath}` command opens a file for editing. It accepts
absolute and relative paths. Tab completion also works for the paths.
The `%` symbol expands to the filepath of the active buffer, `%:h`
__re__ `%%` expands only to the path of the active buffer.

Another method to open files is to used the `:find` command. It searches
for the passed filename in the `path`. The path can be set like 
`:set path+=app/**` (something like this is done automatically by
rails.vim). A `<Tab>` expands the path of the found file and cycles
through the matches.

The `:ls` command shows all loaded buffers in the Buffer list. `%`
indicates the visible buffer and `#` shows the alternate buffer. We can
quickly switch between these two buffers by pressing `<C-^>`. `:bprev`
__re__ `<Leader>b`, `:bnext` __re__ `<Leader>B`, `:bfirst` and `:blast`
move you through the list. `:buffer {number|name}` jumps directly to a
buffer. `:bdelete {number1 number2 ...}` or `:startNumber,endNumber
bdelete` delete the corresponding buffers. A modified file is indicated
with `+`. A command which would switch the buffer raises an error if the
current buffer is modified.  Such a command can be executed without an
error by using the `!` as postfix, like `:bnext!`. Now the modified
files is marked with the letter `h` for hidden. The current buffer is
marked with an `a` for active. If the `hidden` option is set, the `!`
versions of the commands are not necessary. This is also useful for
working with the argument list (`:argdo`). When Vim is closed with `:q`,
it loads the first hidden file, which can be saved with `:w` or
discarded with `:e!`. `:qa[ll]` closes all windows, discarding changes
without warning. `:wa[ll]` writes all modified buffers to disk.

If Vim is started by passing a directory as argument, it starts netrw,
its file explorer. Netrw is also started when passing a directory to
`:edit`. The NERDTree plugin can be used as replacement and is also
started automatically.

A file which is only editable by a super user, can normally not be saved
by Vim, when Vim is not started by a super user. Using the following
command this can be circumvented: `:w !sudo tee % > /dev/null` __re__
`:wsu`.

## Split Windows

The active window can be subdivided with following commands:

| Command            | Description                                                               |
| -------            | -----------                                                               |
| `<C-w>s`           | Split the current window horizontally                                     |
| `:sp[lit] {file}`  | Split the current window horizontally, loading {file} into the new window |
| `<C-w>v`           | Split the current window vertically                                       |
| `:vsp[lit] {file}` | Split the current window vertically, loading {file} into the new window   |
| `<C-w>o`           | Keep only the active window                                               |
| `:on[ly]`          | Keep only the active window                                               |
| `<C-w>c`           | Close the active window                                                   |
| `:cl[ose]`         | Close the active window                                                   |

Navigation between the windows is achieved with:

| Command  | Description                   |
| -------  | -----------                   |
| `<C-w>w` | Cycles to the next window     |
| `<C-w>h` | Focus the window to the left  |
| `<C-w>j` | Focus the window below        |
| `<C-w>k` | Focus the window above        |
| `<C-w>l` | Focus the window to the right |

The size of the windows is changed with the following keystrokes:

| Command       | Description                              |
| -------       | -----------                              |
| `<C-w>=`      | Equalize width and height of all windows |
| `<C-w>_`      | Maximize height of the active window     |
| `<C-w>bar`    | Maximize width of the active window      |
| `[N]<C-w>_`   | Set active window height to [N] rows     |
| `[N]<C-w>bar` | Set active window width to [N] columns   |

`bar` stands for `|`.

## Tabs

Vim also supports tabs to manage multiple files. The following commands
are available:

| Ex Command              | Normal Command      | Descriptions                                               |
| ----------              | --------------      | ------------                                               |
| `:tabnew {filename}`    | __re__ `<C-t>`      | Open {filename} in a new tab or an empty new tab           |
| `:tabe[dit] {filename}` |                     | Open {filename} in a new tab                               |
|                         | `<C-w>T`            | Move the current window into its own tab                   |
| `:tabc[lose]`           |                     | Close the current tab and all of its windows               |
| `:tabo[nly]`            |                     | Keep the active tab page, closing all others               |
| `:tab[ext] {number}`    |                     | Switch to tab page {number}                                |
| `:tab[ext]`             | `gt` __re__ `<S-k>` | Switch to next tab page (right)                            |
| `:tap[revious]`         | `gT` __re__ `<S-j>` | Switch to previous tab page (left)                         |
| `:tapmove {number}`     |                     | Move current tab page to {number}, or end if it is omitted |

`:tabd[o]` is like `:argdo` but executes the given command in the
current window of each tab.

## Search

A search can be initiated in Normal mode with either `/` for forward
searches or `?` for backward searches. `n` moves to the next match in
the initial direction, whereas `N` moves against that direction. `/<CR>`
and `?<CR>` can be used to jump forward or backward to the next or previous
match of the last used pattern. The up and down arrows (`<Up>`,
`<Down>` also custom mapped to `<C-p>` and `<C-n>`) go through the
search history. Highlighting of the search result can be enabled with
`:set hlsearch` or disabled with `:set nohlsearch` (default). `:noh`
mutes the highlighting temporarily until the next search. When
`incsearch` is enabled (default is disabled), Vim highlight matches as
the search pattern is typed and does not wait until `<CR>` is pressed.
If `<Esc>` is pressed during incremental search, the cursor returns to
the starting position. Autocompletion of search patterns can be triggered
with `<C-r><C-w>`. The count of matches of the current pattern is
displayed with the following trick: `:%s///gn`.

After a search the cursor is positioned at the first character of the
match. The cursor can also be placed at the end of the match with the
following search: `/{pattern}/e<CR>` or with the last pattern `//e<CR>`.

Commands can operate on the whole match by using `//e`. So for example
for changing all characters of a match to uppercase this command could
be used `gU//e<CR>`. To repeat this action with the dot command, the next
match must be addressed with `//<CR>` and not with `n`.

If a search pattern needs to be refined it can be reused by typing
`/<Up>`. For elaborated tweaks the Command-Line Window prepopulated with
the search history is useful, which is started with `q/` or `<C-f>` from
Command mode. After the changes are made, any command can be executed
with `<CR>`. The Command-Line Window prepopulated with the last executed
commands is opened with `q:`.

`gn` moves to next search result and uses visual mode to select it. This
can be useful to combine with actions like `dgn`. Also the `.` command
now both moves to the next result and performs the action.

## Patterns

Patterns in Vim can be used to accomplish different tasks, like
searching or substituting.

The case sensitivity can be set globally by enabling the `ignorecase`
setting. It can be overridden per search using `\c` for ignoring the
case and `\C` to be case sensitive. The switch can be placed anywhere
inside a pattern. Another global option is `smartcase` which ignores the
case as long as now uppercase character is in the search pattern.

In the default mode parenthesis `()` must be escaped for grouping. `{}`
are per default also matched literally whereas square brackets `[]` have
special meaning. A pattern for 6 digit hex values like a color in CSS 
would look like:

```
/#\([0-9a-fA-F]\{6}\)
```

Note that the curly brackets only have to be escaped when they are
opened. The very magic search is closer to Perl, Python or Ruby regular
expressions and is enabled with `\v`. Now all characters except '_',
letters and numbers have special meaning. The color example now could be
written as:

```
/\v#([0-9a-fA-F]{6})
```

Using the `\x` character class for hex digits results in:

```
/\v#(\x{6})
```

In other cases it is useful to use verbatim searches. These can be
enabled with the very nomagic switch `\V`. Searching for example for
'e.g.' would be `/e\.g\.` and `/\Ve.g.` with very nomagic mode. In this
mode only `\`, `/` (only for forward searching) and `?` (only for
backward searching) have a special meaning.

Since the escaping of characters by hand can be cumbersome, Vim script
has a function that can be used instead: `escape({string}, {chars})`.
The first argument is the string in which characters should be escaped.
This string could for example be passed by a register with
`@{register}`. An URL which resides in register u could be searched for
like:

```
/                                  // to bring up the search prompt (or ? for backward searches)
/\V                                // to search literally
<C-r>=                             // to go to the expression register prompt
=escape(@u, getcmdtype().'\')<CR>  // escape all '/' and '\'. '.' concatenates strings
```

Parentheses (only in very magic mode with out escaping) can be used to
capture submatches. For example duplicated words can be found with
`\v(\w+)\_s+1\1>`. Captured submatches can be accessed with `\{number}`
where the number starts at 1. `\0` is the entire match. In this
expression the word boundaries are matched with `< >` and `\_s` matches
whitespace or a line break. These submatches are not particularly useful
for search, but are very handy for substitutions.

`\zs` and `\ze` can be used to crop the match and make it to a subset of
the pattern. Example: Search for Vim but only highlight Vi `Vi\zem`.
Another example is matching the content of a string without the
delimiters: `/\v"\zs[^"]+\ze"`.

| Character(s) | Meaning in regular expression                |
| ------------ | -----------------------------                |
| `\w`         | Word characters: letters, numbers and '_'    |
| `\W`         | Everything except word characters            |
| `%(...)`     | Don't capture submatch                       |
| `\_s`        | Matches whitespace or line break             |
| `<...>`      | Word boundaries (do not match any character) |
| `\zs`        | Start of the match (also zero width)         |
| `\ze`        | End of the match (also zero width)           |

<a id="Substitution"></a>
## Substitution

Vim supports powerful substitutions with following ex command:

`:[range]s[ubstitute]/{pattern}/{string}/[flags]`

The flags are explained in the table below:

| Flag | Meaning                                                                         |
| ---- | -------                                                                         |
| `g`  | Globally applies substitution, which means not just the first in each line      |
| `c`  | Substitutions must be confirmed or rejected                                     |
| `n`  | Suppresses the substitution behavior and counts the occurrences of the pattern |
| `i`  | Ignore case for the pattern regardless of the overall settings                  |
| `I`  | Don't ignore case for the pattern regardless of the overall settings            |
| `&`  | Reuses flags from last substitution                                             |
| `e`  | Suppress error messages                                                         |

When searching with the `c` flag the following options are available:

| Option  | Meaning                                          |
| ------  | -------                                          |
| `y`     | Substitute this match                            |
| `n`     | Skip this match                                  |
| `q`     | Quit substituting                                |
| `l`     | "last"; Substitute this match, then quit         |
| `a`     | "all"; Substitute this and any remaining matches |
| `<C-e>` | Scroll the screen up                             |
| `<C-y>` | Scroll the screen down                           |

Special characters for the replacement string:

| Symbol          | Description                                                   |
| ------          | -----------                                                   |
| `\r`            | Insert a carriage return                                      |
| `\t`            | Insert a tab character                                        |
| `\\`            | Insert a backslash character                                  |
| `\{1-9}`        | Insert the first to ninth submatch                           |
| `\0` or `&`     | Insert the entire matched pattern                             |
| `~`             | Use {string} from the previous substitution command execution |
| `\={Vim script} | Evaluate {Vim script} expression and use it as {string}       |

The last search pattern can be used by leaving the `{pattern}` empty.
This allows for decoupling of searching and replacing, so that on a
first attempt with a wrong pattern, the substitution isn't performed on
wrong matches. A drawback of this method is that the two steps go into two different histories and cannot be reused easily. To get a good
history even with a separate search can be achieved by pasting the last
search into the command line with `<C-r>/`.

It is also possible to use the content of registers as `{pattern}` or
`{string}` by using `<C-r>{register}` or by typing `\=@{register}`. The
latter is passing by reference which can take care of line breaks and
special characters. After the `\=` every Vim script is evaluated.

The last substitution can be repeated on the whole file by pressing `g&`
which is equivalent to `:%s//~/&`. `:&` (or `&` in Normal mode) repeats
the last substitution command. If another ampersand is added (`:&&`) the
last flags also are reused.
__re__: `:&&` is mapped to `&` in Normal and Visual mode.

Submatches can be accessed in Vim script with the `submatch({0-9})`
function. For example to increment each HTML heading this feature could
be used like: `/\v/\<\/?h\zs\d/\=submatch(0)+1/g`.

To swap two words (a and b in the example) the substitution command can
be used with a little help of a dictionary in Vim script:

```
/\v(<a>|<b>) // searches for both words
// uses lookup dictionary to find right replacement
:%s//\={"b":a,"a":b}[submatch(1)]/g 

```

An alternative is the Abolish.Vim plugin which reduces the same task to
`:%S/{a,b}/{b,a}/g`.

Substitution in multiple files can be achieved with the standard
commands like in the following example:

```
/Searching     // define the search pattern
:args **/*.txt // use the argument list to collect all files
:set hidden    // so that the files can be edited without saving
:argdo %s//replacement/ge
```

An alternative is to use the Quickfix List (`copen`), `:Vimgrep` and the
qargs plugin or a Vim script (part of
[dotjanus](https://github.com/Christof/dotjanus)).

```
:vimgrep /Searching **/*.txt // search for the pattern
:Qargs                       // load Quickfix list fies as arguments
:argdo %s//replacement/ge  
:argdo update                // to save the files
```

The last three commands could be executed using the bar operator like:
`Qargs | argdo %s/replacement/ge | update`.

## Tools
### ctags

To use ctags it must be installed on the system. It is called
exuberant-ctags on linux. If the program is installed the tags can be
created with `:!ctags -R --exclude=.git --languages=-sql`. This command
creates tags recursively from the current working directory including
the git folder and SQL files. Vim can execute the command automatically
every time after a file is saved: `:autocmd BufWritePost * call
system("ctags -R")`. The tags are stored in a plain text file which is
called tags. Keywords are addressed with a search command. This ensures
that little changes don't require updating of the tags.
Placing the cursor on a word and pressing `<C-]>` __re__ `<Leader>g`
moves to the definition of the word under the cursor. Vim maintains a
history of all jumps. The `<C-t>` shortcut or `:pop` go back in the history. If
a keyword has multiple matches the one with the highest priority (for
example a match in the same file) is used. `g<C-]>` behaves like `<C-]`
except that it show a list of multiple matches instead of moving to the one
with the highest priority. The `:tselect` command can be used to select
another match. `:tnext`, `:tprev`, `:tfirst` and `:tlast` can also be
used to move between matches. The Ex commands `:tag {keyword}` and
`:tjump {keyword}` are the same as `<C-]>` and `g<C-]>`, respectively.
These two commands commands support tab completion as well as searching
which is achieved by replacing `{keyword}` with `/{pattern}`.

### Quickfix List

Source code can be compiled within Vim by issuing the `:make` command.
Vim records all warnings and errors in the quickfix list and jumps to
the first item in the list. If one wants to stay at the current location
the `:make!` should be used instead. The following list of command moves
through the quickfix list (and through the location list if the prefix
`c` is replaced with an `l`):

| Command   | Action                                                         |
| -------   | ------                                                         |
| `:cnext`  | Jump to next item                                              |
| `:cprev`  | Jump to previous item                                          |
| `:cfirst` | Jump to first item                                             |
| `:clast`  | Jump to last item                                              |
| `:cnfile` | Jump to first item in next file                                |
| `:cpfile` | Jump to last item in previous file                             |
| `:cc N`   | Jump to nth item                                               |
| `:copen`  | Jump the quickfix window                                       |
| `:cclose` | Close the quickfix window (even when other window is active)   |
| `:colder` | Older version of quickfix list (can be prefixed with a number) |
| `:cnewer` | Newer version of quickfix list (can be prefixed with a number) |

`:cnext` and `:cprev` can be prefixed by a number to skip entries. The
quickfix list can be traversed like any other text with `j` and `k` as
well as with search commands. Pressing `<CR>` on an entry opens the
corresponding location in the window above. Vim stores old quickfix lists.
The history can be accessed with `:colder` and `:cnewer`. This enables
reusing of results of `:make` and `:grep`.

#### Location list

For every command which populates the quickfix list (`:make`, `:grep`
and `:Vimgrep`) there is a corresponding command with the prefix `l`
(`:lmake`, `:lgrep` and `:lVimgrep`) which populates the location list.
The difference between the two lists is that there can only exist one
quickfix list but many location lists. The location list is bound to the
active window.

#### Customizing make

Vim can execute other 'compilers' instead of make. For example for
javascript one could execute
[nodelint](https://github.com/tav/nodelint). The program which is
called when `:make` is invoked can be set with `makeprg` option. For
nodelint this would be `:setlocal makeprg=NODE_DISABLE_COLORS=1\
nodelint\ %`. The output can be parsed to populate the quickfix list.
The option `errorformat` defines how the output is parsed. The
`:compiler {name}` configures a compiler.

### Project-wide search

`:grep` is a wrapper around grep (or any program like grep, e.g. ack) to
stay in Vim when searching multiple files. The results are placed in the
quickfix list. For example searching for a word ignoring the casing can
be achieved with `:grep -i search_word *`. The asterisk denotes the file
types which should be considered. As with make the executed program can
be change. The options used to configure the behavior are `greprg` and
`grepformat`. As another alternative Vim provides the `:Vimgrep` command
which uses the internal search engine: `:vim[grep][!] /{pattern}/[g][j]
{file(s)}`. The `j` option prevents Vim from jumping to the first match.
For the file(s) a wildcard can be used. `**` stands for any file in the
directory or in subdirectories. `##` expands to the files in the
argument list. Leaving the search field empty to use the last search
like for `:substitute` and `:global` is not supported.

### Autocompletion

Autocompletion can be triggered in insert mode with `<C-n>` and `<C-p>`
which move through the list of suggested items (next and previous). Case
sensitivity follows the setting of the search command. The `infercase`
option can be set, so that the casing is inferred from the typed
characters. Vim supports multiple completion types:

| Command      | Type of completion      |
| -------      | ------------------      |
| `<C-n>`      | Generic keywords        |
| `<C-x><C-n>` | Current buffer keywords |
| `<C-x><C-i>` | Included file keywords  |
| `<C-x><C-]>` | Tags file keywords      |
| `<C-x><C-k>` | Dictionary lookup       |
| `<C-x><C-l>` | Whole line completion   |
| `<C-x><C-f>` | Filename completion     |
| `<C-x><C-o>` | Omni-completion         |

Independent from the type of completion the list of suggestions can be
used with the following commands:

| Command         | Description                                  |
| -------         | -----------                                  |
| `<C-n>`         | Use next item                                |
| `<C-p>`         | Use previous item                            |
| `<Down>`        | Select next item                             |
| `<Up>`          | Select previous item                         |
| `<C-y>`, `<CR>` | Accept currently selected item (yes)         |
| `<C-e>`         | Revert to originally typed text (exit popup) |
| `<BS>`, `<C-h>` | Delete on character from the match           |
| `<C-l>`         | Add on character from current match          |
| `{char}`        | Stop completion and insert {char}            |

Using `<C-n><C-p>` opens the autocompletion popup without using a
match so that the list can be narrowed down by typing more characters.

To use the dictionary lookup the internal spell checker must be enabled
(`:set spell`) or the `dictionary` option must be set to a file or many
files containing word lists.

The line completion ignores the indentation at the start of the line, so
it is useful for duplicating lines when we don't know where the source
line is.

Filename autocompletion always expands files relative to the working
directory and not to the currently edited file. The working directory
can be queried with `:pwd` and changed with `:cd {path}`, `:cd -`
changes back to the last directory.

Omni-completion (`<C-x><C-o>`) is Vim's intellisense. It gives choices
depending on the context and therefore is dependent on file-type
plugins.

### Spell Checker

Vim has a built in spell checker. It is enabled with `:set spell`. The
following commands are available to move between unknown words and to
interact with the dictionary:

| Command | Description                                  |
| ------- | -----------                                  |
| `[s`    | Jump to next spelling error                  |
| `]s`    | Jump to previous spelling error              |
| `z=`    | Suggest corrections for current word         |
| `zg`    | Add current word to spell file               |
| `zw`    | Remove current word from spell file          |
| `zug`   | Revert `zg` or `zw` command for current word |

The default spell checking language is English (`en`). This can be changed with
the `spelllang` option, which is specific to a buffer. The default `en`
supports different variants of English and permits the different
spellings. This can be narrowed down by setting it to `en_us`. More
details can be found under `:h spell-remarks`. If the option is set to a
language which is not available locally, downloading of the language
file is offered automatically. When adding words with `zg` they are put
into the default spellfile. Vim supports multiple spellfiles, which can
be handy for organizing different groups of words. For example a file
containing Vim jargon can separate Vim commands from normal text. To add
a spellfile use `set spellfile+=/path/to/spellfile.utf-8.add`. Now the
file which should store the new word is given by prepending the command
with the number: `2zg` would add it to the new spellfile if only one was
previously set.

Spell checking also works in Insert mode with the special autocompletion
command `<C-x>s`. This goes back in the current line to a misspelled
word and shows options in a popup menu.

### Customization

Vim can be customized by changing settings in the `.vimrc` file or
setting it for the current session with an Ex command. The list of
options is shown with `:h option-list`. For boolean options like
`ignorecase` the following operations can be performed:

* `:set ignorecase` enabled the feature
* `:set noignorecase` disables the feature
* `:set ignorecase!` toggles the setting
* `:set ignorecase?` returns the current value of the option
* `:set ignorecase&` reset the option to the default value

Settings with values can be changed like: `:set tabstop=2`.
`:setlocal` can be used instead of `:set` to change the settings only
for the active buffer or window (for example for `number`).

Changes can be written es Ex commands in any file. The file can be
loaded with `:source {filename}` which executes each line as Ex command.
The `.vimrc` file is opened with the command `:edit $MYVIMRC`. After
performing changes in the file the system can be updated with 
`:source $MYVIMRC`. If the file is in the active buffer then `:so %` can
be used as shortcut.
 
To change settings depending on the file type the `autocmd` can be used,
which listens to events, like file types. To change the tabbing for ruby
the following command can be used: `autocmd FileType ruby setlocal
tabstop=2`. The file type option must be enabled so that the command can
work: `filetype on`. An alternative which is useful for many changes is the
`ftplugin`. This loads whole files (residing in the `ftplugin` directory
of the `.vim` folder) depending on the file type. It is enabled with
`filetype plugin on`.

