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
