neovim
======
- Category: Learning
- Tags: 
- Created: 2025-06-30T18:23:34-07:00

## LESSON 1

### 1.1 - Moving the cursor

- Move the cursor with `h`, `j`, `k`, `l`
	- `k` - Up
	- `j` - Down
	- `h` - Left
	- `l` - Right

### 1.2 - Exiting Neovim

- Follow these steps:
	- Press `<Esc>` to get you into Normal mode
	- Type `:q!`
		- `:` - begin command
		- `q` - exit file
		- `!` - DISCARD any changes you have made

### 1.3 - Text editing - Deletion

- Press `x` to delete the character under the cursor in Normal mode

### 1.4 - Text editing - Insertion

- Press `i` to enter Insert mode
- Press `Esc` to return to Normal mode

### 1.5 - Text Editing: Appending

- Press `A` to append text
	- Enters Insert mode and places you at the end of the line

### 1.6 - Editing a file

- Use `:wq` to write a file and quit

## LESSON 2

### 2.1 - Deletion commands

- Type `dw` in Normal word to delete the word over which your cursor rests

### 2.2 - More deletion commands

- Type `d$` to delete to the end of the line
	- Goes from the immediate cursor lcoation to the end of line *inclusive*, not on a word basis

### 2.3 - Operators and motions

- Many commands are made from a **command** and a **motion**.
	- Command - what do I do?
	- Motion - until when do I do it?
	- Example: `dw` = (d)elete until the start of the next (w)ord
- Short list of motions
	- `w` - Until the start of the next word, *excluding* the first character
	- `e` - Until the end of the current word, *including* the last character
	- `$` - Until the end of the line, *including* the last character
- Typing a motion in Normal mode performs it without an operation. 
	- For example, w moves you to the start of the next word

### 2.4 - Using a count for a motion

- Typing in a number before a motion repeats it that many times
- Type `0` to move to the beginning of the line

### 2.5 - Using a count to delete more

- Typing a number with an operator repeats it that many times
	- Example - `d2w` deletes two words forward from the current cursor

### 2.6 - Operating on lines

- Type `dd` to delete whole words
	- Numebrs also work before dd to delete multiple lines at once

### 2.7 - The Undo command

- Type `u` to undo the last commands, `U` to fix a whole line
- Type `Ctrl-R` to redo commands

