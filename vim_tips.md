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

| Command                       | Jumps to                                        |
| -------                       | ------                                          |
| `[line]G`                     | specified line                                  |
| /{pattern}                    | pattern                                         |
| `%`                           | matching parenthesis                            |
| `(`/`)`                       | start of previous/next sentence                 |
| `{`/`}`                       | start of previous/next paragraph                |
| `H`/`M`/`L`                   | start of top/middle/bottom of screen            |
| `gf`                          | file name under the cursor (suffixesadd option) |
| `<C-]>` _re_`<Leader>g`       | definition of keyword und der cursor            |
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
if a search command finds nothing on the line). Commands can be added to
a macro if the uppercase letter is used for the register.

Macros can be edited directly in vim as text by putting the content of
the register into the file. This can be achieved with `"{register}p` or
with `:put {register}`. The later always puts the content into a new
lines below the current line, which is preferable for editing a macro.
To get the edited text back into the register `"{register}dd` or (`:da`)
can be used. This could cause problems because it appends a trailing
`^J` for the linebreak. To avoid this, this longer version can be used:
`0"{register}y$dd`. An alternative to editing the macro in the text is
to use Vim script functions like `substitute()`. A list of functions can
be found in the help files: `:h function-list`. Subsititue can be used
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
:let i = 1           // declare a vim script variable i
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
the starting position. Autocompletion of search patterns can be triggerd
with `<C-r><C-w>`. The count of matches of the current pattern is
displayed with the following trick: `:%s///gn`.

After a search the cursor is positioned at the first character of the
match. The cursor can also be placed at the end of the match with the
following search: `/{pattern}/e<CR>` or with the last pattern `//e<CR>`.

Commands can operate on the whole match by using `//e`. So for example
for changing all characters of a match to uppercase this command could
be used `gU//e<CR>`. To repeat this action with the dot commad, the next
match must be addressed with `//<CR>` and not with `n`.

If a search pattern needs to be refined it can be reused by typing
`/<Up>`. For elaborated tweaks the Command-Line Window prepopulated with
the search history is useful, which is started with `q/` or `<C-f>` from
Command mode. After the changes are made, any command can be executed
with `<CR>`. The Command-Line Window prepopulated with the last executed
commands is opened with `q:`.

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
special meaning. A pattern for 6 digit hex values like a color in css 
would look like:

```
/#\([0-9a-fA-F]\{6}\)
```

Note that the curly bracktes only have to be escaped when they are
opened. The very magic search is closer to Perl, Pyhton or Ruby regular
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
enabeld with the very nomagic switch `\V`. Searching for example for
'e.g.' would be `/e\.g\.` and `/\Ve.g.` with very nomagic mode. In this
mode only `\`, `/` (only for forward searching) and `?` (only for
backward searching) have a special meaning.

Since the escaping of characters by hand can be cumbersome, Vim script
has a function that can be used instead: `escape({string}, {chars})`.
The first argument is the string in which characters should be escaped.
This string could for example be passed by a register with
`@{register}`. An URl which resides in register u could be searched for
like:

```
/                                  // to bring up the search prompt (or ? for backward searches)
/\V                                // to search literally
<C-r>=                             // to go to the expression register prompt
=escape(@u, getcmdtype().'\')<CR>  // escape all '/' and '\'. '.' concatenates strings
```

Parantheses (only in very magic mode with out escaping) can be used to
capture submatches. For example duplicated words can be found with
`\v(\w+)\_s+1\1>`. Captured submatches can be accessed with `\{number}`
where the number starts at 1. `\0` is the entire match. In this
expression the word boundaries are matched with `< >` and `\_s` matches
whitespace or a linebreak. These submatches are not particularly useful
for serach, but are very handy for substitutions.

`\zs` and `\ze` can be used to crop the match and make it to a subset of
the pattern. Example: Seach for Vim but only highlight Vi `Vi\zem`.
Another example is matching the content of a string without the
delimiters: `/\v"\zs[^"]+\ze"`.

| Character(s) | Meaning in regular expression                |
| ------------ | -----------------------------                |
| `\w`         | Word characters: letters, numbers and '_'    |
| `\W`         | Everything except word characters            |
| `%(...)`     | Don't capture submatch                       |
| `\_s`        | Matches whitespace or linebreak              |
| `<...>`      | Word boundaries (do not match any character) |
| `\zs`        | Start of the match (also zero width)         |
| `\ze`        | End of the match (also zero width)           |

## Substitution

Vim supports powerful substituions with following ex command:

`:[range]s[ubstitute]/{pattern}/{string}/[flags]`

The flags are explained in the table below:

| Flag | Meaning                                                                         |
| ---- | -------                                                                         |
| `g`  | Globally applies substitition, which means not just the first in each line      |
| `c`  | Substitutions must be confirmed or rejected                                     |
| `n`  | Suppresses the substitution behaviour and counts the occurrences of the pattern |
| `i`  | Ignore case for the pattern regardless of the overall settings                  |
| `I`  | Don't ignore case for the pattern regardless of the overall settings            |
| `&`  | Reuses flags from last substitution                                             |

When searching with the `c` flag the following options are available:

| Option  | Meaning                                              |
| ------  | -------                                              |
| `y`     | Substitute this match                                |
| `n`     | Skip this match                                      |
| `q`     | Quit substituting                                    |
| `l`     | "last"; Substitute this match, then quit             |
| `a`     | "all"; Substitute this and and any remaining matches |
| `<C-e>` | Scroll the screen up                                 |
| `<C-y>` | Scroll the screen down                               |

Special characters for the replacement string:

| Symbol          | Description                                                   |
| ------          | -----------                                                   |
| `\r`            | Insert a carriage return                                      |
| `\t`            | Insert a tab character                                        |
| `\\`            | Insert a backslash character                                  |
| `\{1-9}`        | Insert the first to nineth submatch                           |
| `\0` or `&`     | Insert the entire matched pattern                             |
| `~`             | Use {string} from the previous substitution command execution |
| `\={Vim script} | Evaluate {Vim script} expression and use it as {string}       |

The last search pattern can be used by leaving the `{pattern}` empty.
This allows for decoupling of searching and replacing, so that on a
first attempt with a wrong pattern, the substitution isn't performed on
wrong matches. A drawback of this method is that the two steps go into
two different histories and cannot be reused easily. To get a good
history even with a separate search can be achieved by pasting the last
search into the command line with `<C-r>/`.
