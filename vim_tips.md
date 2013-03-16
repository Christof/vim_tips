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

`:reg "{register}` shows the content of the register or all registers
if no register is given.

Some more special registers:
| Register | Content/Function                                |
| -------- | ----------------                                |
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

### Little things
#### Arithmetic
`<C-a>` adds `[count]` to the number at or after the cursor, `<C-x>`
subtracts. If no number is given the default is 1.
