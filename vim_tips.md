# Vim tips
## Normal mode
### Motions
#### Search
The search command is exclusive so you can use it in `d{motion}`
commands and search for the first word which should not be deleted:

	This is an example
	d/an<CR>
	an example

The cursor ends up at a.

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

Those commands are not really motions, but can be used in visual and operator pending mode.
