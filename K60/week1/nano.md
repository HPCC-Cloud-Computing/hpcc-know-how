# Nano editor on Linux

## Description

Nano is a small friendly editor with several cool features, such as opening multiple files, scrolling per line, undo/redo, syntax coloring, line numbering, and soft-wrapping overlong lines.

Open nano on terminal by the following syntax:

    nano [options] [[+line[, column]] file]...


while (line, column) is the cursor position of the opening file and common options are in the table below.

Option  |  Usage
-- |--
-B, --backup  |  Store the previous version of the file on saving. The backup file name is the current file name with a '~' suffix.
-C `dir`, --backupdir=`dir` | Store multiple backup version in `dir`.
-E, --tabtospace  |  Convert tabs to spaces.
-c, --constantsow  | Constantly show the cursor position on the status bar.
-i, --autoindent  |  Indent new line automatically.
-l, --linenumbers  |  Display line numbers to the left of the text area.
-r `number`, --fill `number`  |  Hard-wrap lines at column `number`.
-v, --view  |  Read-only mode.

## Nano commands

`Ctrl-key` sequences are notated with a `^`

`Meta-key` sequences are notated with `M-` and can be enterd by using `Alt` key.

This is a list of command nano commands

Command  |  Usage
--  |  --
^G  |  Display this help text
^X  |  Close the current file buffer / Exit from nano
^O  |  Write the current file to disk
^R  |  Insert another file into the current one
^W  |  Search forward for a string or a regular expression
^\  |  Replace a string or a regular expression
^K  |  Cut the current line and store it in the cutbuffer
^U  |  Uncut from the cutbuffer into the current line
^J  |  Justify the current paragraph
^T  |  Invoke the spell checker/ linter/ formatter, if available
^C  |  Display the position of the cursor
^_  |  Go to line and column number
M-U  |  Undo the last operation
M-E  |  Redo the last undone operation
M-6  |  Copy the current line and store it in the cutbuffer
M-]  |  Go to the matching bracket
M-W  |  Repeat the last search
M-▲  |  Search next occurrence backward
M-▼  |  Search next occurrence forward

## Editting

### Moving the cursor
You can enter text  and moving around in a file by the normal cursor movement keys.

Movement  |  Meaning
--  |  --
left arrow  |  Move to the character in the left
right arrow  | Move to the character in the right
up arrow  |  Move to the line above
down arrow  |  Move to the line below
`Ctrl` + left arrow  |  Move to the word in the left
`Ctrl` + right arrow  | Move to the word in the right
`Ctrl` + up arrow  |  Move to the paragraph above
`Ctrl` + down arrow  |  Move to the paragraph below

### Cut and paste

Type `^K` to delete the current line and put it in the cutbuffer, then you can paste it to the current cursor position by typing `^U`.

### Select text from keyboard

Start selecting by holding `Shift` and moving the cursor around. Copy selected characters by pressing `Alt` + `6`, or cut them using `Ctrl` + `K`, or delete them with the `Delete` button.
