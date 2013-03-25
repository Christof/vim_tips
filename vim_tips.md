# Vim tips
## Notes
In all code examples * denotes the cursor position.
_re_ after a command and in front of another command means that the default command which is the first one is remapped to the second, e.g. `<C-]>` _re_`<C-j>`.

## Normal mode
### Shortcuts

| Longhand | Compound | Descriptions                                               |
| -------- | -------- | ------------                                               |
| `cl`     | `s`      | Replaces the underlying character but stays in insert mode |
| `^C`     | `S`      | Changes the whole line independent of cursor positon       |

### Copy & Paste

Vim's copy command is called yank `y`, past is `p` and cut is `d`. They
use the unnamed register `""` if not specified otherwise. A specific
register can be used be preceeding the command with `"{register}"`.
A register name can be a character from `a` to `z`. If a register is
addressed with an uppercase letter the content is appended to it. On the
other hand lowercase letters overwrite the content of the register.
Another special register is the black hole register `"_` which doesn't
store anything. This could also be a used in the example below.

A yank does not only put the content into the unnamed register but also
in the yank register `"0`. This is helpfull when replacing something
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
before the cursor. Depending on the yankend content, before and after
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
because vim treats the in put as if someone was typing it and applys
intendation rules. To disable autoindent, abbreviations and other
features which are not usefull when pasting, the `paste` option can be
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
| `gU`    | Make upercase  |
| `>`     | Shift right    |
| `<`     | Shift left     |

### Motions
#### Search
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
any kind of paranthesis or `"` and `'`, like text-objects.

| Old text         | Command | New text         | Description               |
| -------          | ------- | --------         | -----------               |
| `Hello w*orld`   | `ysiw)` | `Hello (world)`  | Add sourroundings `ys`    |
| `"Hello *world"` | `cs"'`  | `'*Hello world'` | Change sourroundings `cs` |
| `"Hello *world"` | `ds"`   | `*Hello world`   | Delete sourroundings `ds` |

#### Jump between matching paranthesis

`%` jumps between matching paranthesis like `()`, `{}` and `[]`, but it
works also for XML tags and language constracts like `do end` and `if
end` in ruby when the _matchit_ plugin is enabled.

#### Jump list

`<C-o>` is like back in a browser and `<C-i>` like forward. `:jumps`
shows the jump list. Jumps are generally larger as single character
or word motions and can move between files. Some jump commands
are listed below:

| Command                  | Jumps to                                        |
| -------                  | ------                                          |
| `[line]G`                | specified line                                  |
| /{pattern}               | pattern                                         |
| `%`                      | matching parenthesis                            |
| `(`/`)`                  | start of previous/next sentence                 |
| `{`/`}`                  | start of previous/next paragraph                |
| `H`/`M`/`L`              | start of top/middle/bottom of screen            |
| `gf`                     | file name under the cursor (suffixesadd option) |
| `<C-]>` _re_`<C-j>`      | definition of keyword und der cursor            |
| `` `{mark}`` _re_`<C-m>` | a mark                                          |

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
the file the upercase letters are global. `´{mark}` move to the line of
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

A macro can be recoreded by pressing `q{register}`. Recording is
indicated in the status line. By typing `q` again the recording is
stopped. The macro can be displayed with `:reg {register}`. `^[` is used
to indicate pressing of the escape key. A macro can be replayed with
`[count]@{register}`. The last macro can be repeated with `@@`. Playback is
aborted if one of the commands fails. This can be thought of as serial
execution. Parallel execution can be achieved with a visual selection
(using probably line-wise Visual mode) and then executing `'<,'>normal
@{register}`. This jumps over lines which produce an error (for example
if a search command finds nothing on the line.)

### Little things
#### Arithmetic
`<C-a>` adds `[count]` to the number at or after the cursor, `<C-x>`
subtracts. If no number is given the default is 1.

## Insert mode

| Shortcut               | Description                              |
| --------               | -----------                              |
| `<C-h>`                | Same as backspace                        |
| `<C-w>`                | Delete back one word                     |
| `<C-u>`                | Delete back to start of line             |
| `<C-o>`                | Switch to Insert Normal mode             |
| `<C-r>{register}`      | Paste from register                      |
| `<C-r><C-p>{register}` | Paste from register and fixe indentation |
| `<C-r>=`               | Evaluate expression (Calculator)         |
| `<C-T>`                | Insert one shiftwidth of indent          |
| `<C-D>`                | Delete one shiftwidth of indent          |
| `<C-v>{code}`          | Enter character by decimal code          |
| `<C-v>u{code}`         | Enter character by hexadecimal code      |
| `<C-v>{nondigit}`      | Enter nondigit literally                 |
| `<C-k>{char1}{char2}`  | Enter character by digraph               |

Insert Normal mode can be used to trigger one normal mode command, after
which Vim returns automatically to insert mode. For example `<C-o>zz`
centers the screen vertically and can be executed during writting in
insert mode.

Some normal mode commands which goe along the above shown shortcuts

| Command     | Description                          |
| -------     | -----------                          |
| `ga`        | Show code for character under cursor |
| `:digraphs` | Sho a table of digraphs              |

### Replace mode

Replace mode can be issued from Normal mode with `R`. It behaves like
other editor when the insert key was presed. Tab characters can pose a
problem in Replace mode if the mode is started on one of the 'spaces'
representing the tab. An alternative is the Virtual Replace mode which
handles the tab character like some space characters and is triggerde
with `gR`.


## Visual mode

Visual mode can be used to select text visually like in other editors
when selecting text with the mouse. It is triggerd with `v` followed by
movement commands. In Visual mode the free end is indicated by the
blinking cursor. The free end can be switched between the start of the
selection and the end of the selection by pressing `o`.

`V` starts the line-wise Visual mode. This can for example be used for
line-wise visual command wich can also be repeated: `Vj>.` selects the
current and next line and indents them twice.

In gernal operators should be prefered to Visual commands if possible,
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
