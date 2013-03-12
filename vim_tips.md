# Vim tips
## Notes
In all code examples * denotes the cursor position.
## Normal mode
### Shortcuts

| Longhand | Compound | Descriptions                                               |
| -------- | -------- | ------------                                               |
| `cl`     | `s`      | Replaces the underlying character but stays in insert mode |
| `^C`     | `S`      | Changes the whole line independent of cursor positon       |

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

#### Markers

To mark a location use `m{a-zA-Z}` the lowercase letters are local to
the file the upercase letters are global. `´{mark}` move to the linke of
the mark. `` `{mark}`` moves to the exact position. Vim defines some
special markers:

| Shortcut | Description                                      |
| -------- | -----------                                      |
| `` `` `` | Poition before the last jump within current file |
| `` `. `` | Location of last change                          |
| `` `^ `` | Location of last insertion                       |
| `` `[ `` | Start of last change or yank                     |
| `` `] `` | End of last change or yank                       |
| `` `< `` | Start of last visual selection                   |
| `` `> `` | End of last visual selection                     |

#### Jump between matching paranthesis

`%` jumps between matching paranthesis like `()`, `{}` and `[]`, but it
works also for XML tags and language constracts like `do end` and `if
end` in ruby when the _matchit_ plugin is enabled.

### Little things
#### Arithmetic
`<C-a>` adds `[count]` to the number at or after the cursor, `<C-x>`
subtracts. If no number is given the default is 1.
