# Global tricks

Last time we introduced global and combined it with `print`, `delete` and `substitute` to get some useful functionality.

Today we'll do more exotic examples inspired by [this article](https://vim.fandom.com/wiki/Power_of_g) from the vim wiki.

# Exercises

## Exercise 1

Insert a blank line after every line in the block below that ends with a period or comma.

There is an ex command `put` which pastes text from the specified register after its target line.

For example this will paste "Boban" after line 10:

- `:let @a = 'Boban`
- `:10 put a`

We can use this mighty command to solve our problem:

- move your cursor to "space" below
- do `:let @a = ''`
- do `:.,+4 g/\v[.,]$/put a`
- hit `u` to undo so we can do it even more directly
- do `:.,+4 g/\v[.,]$/put _`

```
space
me,
out
man.
whoaa.
```

Breaking down that first command:

- `.,+4` - a range from this line to 4 lines down
- `g` - start a global command
- `\v[.,]$` - the pattern
    - `\v` - very magic flag to make sure square brackets and dollar are magic
    - `[.,]` - period or comma
    - `$` - end of the line
- `put a` - put what's in register `a` (an empty line) on the next line

The second command is the same except it uses the "black hole" register `_` - see [kata 19](019_special_registers.md).
The blackhole register is always "empty", so if you paste from it, you get an empty line.

This little trick with the blackhole register means we don't need to set an empty register beforehand.

## Exercise 2

Duplicate every uncommented line in the block below.

For this one we can just use a normal command to yank the current line and paste below.

- put your cursor on the first line
- do `:.,+4 v/\v^#/normal Yp`

```python
# Comment
a = None
# More comments
def f():
    return None
```

Remember from last time that `v` is short for `vglobal` which applies your ex command to every line
that _doesn't_ match the pattern.

The pattern `\v^#` is the same one from last time which matches any line starting with '#'.

`Yp` is `Y` (yank current line) plus `p` (paste below current line).

You might be wondering why the above didn't cause an infinite loop. For example after it duplicates `a = None`,
won't it then also read that duplicate line in its next step, then duplicate that etc...
The global/vglobal commands thankfully do their processing in 2 passes:

- the first pass determines all the lines to apply the ex command to
- the second pass applies the changes, potentially modifying the buffer

Hence the duplicate lines don't yet exist in that first pass and we avoid these kinds of infinite loops.

## Exercise 3

Move all the uncommented lines from the first block to the empty second block below it.

This time we're wanting to _move_ lines, so we'll need a new ex command: `move` (or `m`).

The `move` command syntax is: `:[RANGE] move [DESTINATION]`
which moves the lines specified by `RANGE` to below `DESTINATION`.
e.g. `:3-5 move 10` moves lines 3-5 below line 10 (ie. line 11).

When you don't specify a range it uses the current line (ie. `.`) which works nicely for us here.

- figure out the line number for the first backtick line in the second block (I'll assume it's 108)
- put your cursor on the first line of python code
- do `:.,+4 v/\v^#/move 108` (put in the line number you found)

```python
# Comment
a = None
# More comments
def f():
    return None
```

```python
```

It might surprise you that this worked - maybe you were expecting the text to be reversed...
Don't worry, we'll hit this bug in the next section!

## Exercise 4

This time we'll _copy_ all the uncommented lines from the first block to the second.

There's a handy `copy` ex command that works like `move`:

- figure out the line number for the first backtick line in the second block (I'll assume it's 132)
- put your cursor on the first line of python code
- do `:.,+4 v/\v^#/copy 132` (put in the line number you found)

```python
# Comment
a = None
# More comments
def f():
    return None
```

```python
```

This time the text was reversed. To understand this, think about each copy being applied sequentially:

- `a = None` gets copied to after line 132
- `def f():` gets copied to after line 132 (so it's now above `a = None` and pushes it down)
- `    return None` gets copied to after line 132 (so it's now above the other 2)

Overall the effect is reversing the lines.

The reason it didn't happen in the `move` exercise, was because we were deleting the line from the original
block which shifted everything below it up 1 line.
This meant that line 108 no longer represented the backtick line, but was tracking the last moved line.

How to fix the reversing?
The issue is that fixed line numbers are too static.
We could introduce `$` which means "the last line in the buffer".
`$` has a dynamic nature where it always represents the last line, even if text has been pasted to the end of the file.

A quick hack would be to just use `$` as the destination, e.g. `[RANGE] v/[PATTERN]/copy $`.
That will append everything in order to the end of the file as a block, then you manually move it in one step
to it's intended destination.

A more complex destination could be something like `$-50` (50 lines from the bottom).
This will still dynamically adjust as the copy commands are run and could save having to do that final manual move.
The tricky thing will be figuring out what the adjustment is
(and it might be faster to just use the first approach as young people nowadays have trouble with basic arithmetic).

Another approach could be to:

- use the command from our exercises
- get a reversed version of your text
- filter it through an external reversing tool (like `tac` if it's installed - `cat` backwards)

You could be also reverse it again by using the same kind of copy command with a static line number.

Note also that you might try shortening `copy` to `c` above - but the shorthand for `copy` is actually `t` :upside-down-face:
For some reason people don't find this intuitive...

# (Optional) Exercise 5

Copy all the uncommented lines into register `a`.

As how `put` copies from a register into the buffer, there is a `yank` ex command for copying from the buffer into a register.

(These are the ex command analogs for `p` and `y` in normal mode)

The `yank` command takes a range and copies it into the register specified,
e.g. `:10-15 yank a` copies lines 10-15 into register `a`.
When the range is missing it uses the current line.

- put your cursor on the first line of python code
- do `.,+4 v/\v^#/yank a`
- do `:echo @a` to see if it worked
- (what? It just says "    return None" - this is because each individual ex command is writing over the register)
- put your cursor on the first line of python code
- do `:let @a = ''` (clearing the register)
- do `.,+4 v/\v^#/yank A`
- do `:echo @a` to see if it worked

```python
# Comment
a = None
# More comments
def f():
    return None
```

The above is incrementally building onto register `a` as each ex command gets processed.

Remember from [kata 18 exercise 4](018_registers.md) that using a capital letter for a register means "append".
Hence `yank A` appends the current line to register `a`.

# Conclusion

As you learn more ex commands, you'll be able to combine them arbitrary search patterns to do some powerful things.

It's worth learning a few of the basic ex commands (like `copy`, `move`, `yank` and `paste`)
to enhance your vim superpowers.
