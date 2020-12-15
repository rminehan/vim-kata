# Reading and writing

When you're editing a file in vim, you can start to think about what you're seeing in your editor _as_ the file.

But in fact vim is just holding a buffer in memory and the file is a separate entity on disk.

Having this idea clear in your head makes it easier to understand the read and write commands.

(And it makes it less confusing when you have buffers that aren't associated with a file on disk)

# Read

The read command is for bringing something external into your buffer, e.g.

- the contents of a file
- the results of a shell command

# Write

Writing is the opposite.

The contents of the buffer are being sent to something external to your buffer, e.g.

- sending it to a file on disk
- using it as standard input for a shell command

## I know write!

You actually use the `write` command all the time when you do `:w` (that's short for `write`).
You're sending the contents of the buffer to the a file on disk associated with your buffer.

This is actually just a special case of a much more general command.

# Exercises

## Exercise 1

The more general form of write is: `:[RANGE] write [DESTINATION]`.

It takes the lines specified by the range and feeds them to a destination "outside" the buffer.

That destination could be a file or a program expecting standard input (analogous to `>` and `|` in the shell).

When the range isn't specified it's the whole buffer.

We'll write the C code below to a file foo.c

- put your cursor on the `int i = 1;` line
- do `:.,+1 write foo.c` which is:
    - `:` - enter command mode
    - `.,+1` - a range meaning
        - `.` - start at the current line
        - `,` - delimiter
        - `+1` - end at the current line + 1 (the line below)
    - `write` - start a write command
    - `foo.c` - file location
- do `:split foo.c` to open foo.c in a split window
- do `:q` to close the split

```c
int i = 1;
int j = 2;
```

## Exercise 2

Let's read the foo.c file back into our buffer with the read command.

The general form for read is `:[DESTINATION] read [SOURCE]`

Destination is the line which the text will be pasted _below_. It default to `.` the current line.

- put your cursor on the top backtick line
- do `:. read foo.c`
- do `:!rm foo.c` to get rid of foo.c (we haven't covered this yet - just roll with it)

```c
```

## Exercise 3

So far our examples have just been using files as source/destination for IO.

If a destination has a `!` before it, it's understood to be a shell program, not a file.

Let's read the contents of the current directory into the current buffer:

- move your cursor to the top backtick line
- do `:. read !ls`
- (that should have generated a list of files from the current directory into the code block)
- press `u` to undo
- put your cursor on the bottom backtick line
- do `:-1 read !ls`
    - here the -1 means "we paste below the line one line above the current line"
    - ie. we paste _on_ the current line

```
```

## Exercise 4

Let's search our text below for lines containing "of".

We'll "write" the text block out through an external grep command:

- move your cursor onto the paragraph of text
- do `vip` to visually select the paragraph
- do `:` (this will pre-populate it with a range)
- add `write !grep of`
- (the results should get printed out in vim's output window)

```

I don't know of him
foo
olive
officer officer!
coffin

```

# Confusion with filtering?

Filtering is a bit of read and write.

It takes a range and feeds that through an external program.

Then it reads the standard output and replaces the original range in the buffer used. 

Like a closed loop.

Sometimes you don't want to replace your buffer contents though,
e.g. you just want to print something in the output window.

Or you want to read in output from a program that doesn't need standard input (like ls).

# Summary

Read and write are very powerful, general purpose commands.

Like filtering, it allows us to work more closely with the shell which reinforces our knowledge of common cli tools.
