# The Global Command

The global command is a very powerful utility that lets you apply broad sweeping changes to a block of text,
with more control over the rules for which lines to apply the change to.

# Motivation

Way back in [kata 3](003_normal_commands) we learnt how to use the normal command against a range of lines.

e.g. `:1,20 normal 0x` deletes the first character of lines 1-20.

The context could be that it's commented out python code that we want to uncomment by deleting the '#'
at the start of the line.

```python
#a = None
#toUpper(a)
b = None
#...
```

Above though not all lines are commented out.
That normal command would delete the 'b' at the front of line 3 because it runs against every line in the range.

What we want is logic like:

> Across lines 1-20, delete the first character if the line starts with '#'

So we need a mechanism to express this "if" logic....

# Introducing the global command

The syntax for the global command is:

```
:[RANGE] global/[PATTERN]/[EX COMMAND]
```

It finds all lines in the range that match the pattern, and applies the ex command to those lines.

# Back to our example

We could represent our desired change with:

```
:1,20 global/\v^#/normal 0x
```

Breaking this down:

- `1,20` - range for lines 1-20
- `global` - start a global command
- `/` - start the pattern match
- `\v^#` - the pattern which is:
    - `\v` - very magic mode (probably not needed here, just for completeness)
    - `^` - line start anchor
    - `#` - hash character
- `/` - start ex command
- `normal 0x` - normal command as above
    - `0` - go to start of line
    - `x` - delete the character under the cursor

So we captured the "if" logic by introducing a pattern which matches just those lines starting with a '#'.

# Shortening the global command

- you can shorten "global" to just "g"

- you don't need spaces between the range and "global" - that's just for readability

- when the range is left out, it defaults to the entire buffer (`%`)

- a lot of the ex commands themselves have shortened forms (e.g. `p` for `print`, `d` for `delete`).

So if we wanted to run the our example across the whole buffer we could do:

```
:g/\v^#/normal 0x
```

# Exercises

## Exercise 1 - printing

We want to print all the lines in the block below that contain "Boban".

Vim comes with an ex command "print" which fits this task nicely.
Print is a simple little command for printing what goes into it to the output window (similar to `echo`).

- move your cursor to the first line ("Boban says this")
- do `:.,+4 global/Boban/print<enter>`
- (this will show you the output window and prompt you to hit Enter)
- (the previous search picked up "Bobanita" lines - let's add word boundaries)
- do `:.,+4 global/\v<Boban>/print<enter>` then Enter again when you've inspected the output
    - remember the `\v` is for "very magic" mode
    - `<` and `>` are left and right word boundaries
    - the left boundary prevents "McBoban" getting matched
    - the right boundary prevents "Bobanita" getting matched
- (we can shorten this a bit)
- do `:.,+4 g/\v<Boban>/p<enter>` and Enter again
    - (we shortened `print` to `p` here)

```
Boban says this
Bobanita says that
That crazy Boban
Son of old McBoban
He eats Bobans for breakfast
```

## Exercise 2 - deleting

Delete all the commented out lines in the code block below.

Translating this to a global command we have:

> For the range of lines, if the line starts with '#', delete it

There's a `delete` ex command we can use to delete lines.

- put your cursor anywhere in the code block
- do `vip` to visually select the paragraph
- do `:` to start a command - note how vim hopefully puts in a range for you
- enter `global/\v^#/delete<enter>`
- press `u` so that we can do it again using a shorter form
- do `gv` to recover your last visual selection
- do `:` to start a command
- enter `g/\v^#/d<enter>`

```python
#a = None
#toUpper(a)
b = None
c = b
#toLower(a)
```

Up until this point in our katas, when I've wanted to use paragraphy operations I've had
to add some blank lines above and below the code block so that the triple backticks don't get included
in the paragraph.

In the example above the backtick lines were included in the visual selection but because they didn't match
the pattern (lines starting with '#') it didn't matter so I didn't include the usual padding.

We'll see though for the `vglobal` exercise that including the backticks would be bad.

## Exercise 3 - vglobal

`vglobal` is like `global` except it applies the ex command to all the lines that _don't_ match the pattern.

Use this immense power to comment out all the lines in the block below that aren't already commented.

Translating this to a vglobal statement it's:

> For the range of lines, if the line doesn't start with '#', add a comment to it

- put your cursor on the first line
- do `:.,+4 vglobal/\v^#/normal I#<enter>`
- hit `u` to undo so that we shorten it a little
- do `:.,+4 v/\v^#/normal I#<enter>`
    - (we just changed `vglobal` to `v`)

```python
#a = None
#toUpper(a)
b = None
c = b
#toLower(a)
```

Note above if we'd used `vip` then `:` to get our range, it would have included the backtick lines.
The backtick lines _don't_ start with '#' so the vglobal command would have commented them out.

## Exercise 4 - substitute

Rename "boban" to "bobanita", but only for lines that aren't commented out.

- put your cursor on the first line
- do `:.,+4 vglobal/\v^#/s/boban/bobanita/g<enter>`

```python
#boban = None
#toUpper(boban)
boban = 1
c = boban
#toLower(boban)
```

So many forward slashes! Hopefully this makes it clearer

```
.,+4 vglobal/\v^#/s/boban/bobanita/g
             ---- ------------------
          vglobal     ex command
          pattern
```

It's confusing because there's two levels of search.
The vglobal does the first pass picking out just the lines not starting with '#'.

Then the substitute command is run on those lines replacing "boban" with "bobanita".

Note in this case our simple search term "boban" was okay, but if this is a variable name in code,
then you'd want to do a case sensitive match to not pick up any "Bobans", "BOBANS" etc...
You can do this by adding `\C` to your search - see [kata 11](./011_buffer_search.md).

# History lesson

## That funny old grep

In exercise 1 we used the `print` command to send the matched lines to vim's version of standard out - the output window.

Suppose we wanted to get a print out of all the lines in a buffer that contained "Boban", then we could do:

```
:g/Boban/p
```

Here we didn't specify a range, so the entire buffer is used by default.

If we were doing this on the command line, we'd used grep, e.g. `cat file | grep Boban`.

These are two ways of doing the same thing.

In general if you want to grep your buffer some regular expression ("re") in vim, you'd do:

```
:g/re/p
```

W-w-w-what? That looks familiar!

This is in fact where grep got it's funny name.
It's a special case of the more general global operator where the ex command is printing.

## And vglobal

It's odd how `-v` usually means something like "version" or "verbose" for most cli tools,
but in grep it has the effect of reversing the pattern,

e.g. the following will find all the lines that _don't_ contain "boban":

```
cat file | grep -v boban
```

Could it be that this `-v` was inspired by the `v` from `vglobal`? (Or vice versa)

# Summary

Global is very powerful - it combines arbitrary patterns with arbitrary ex commands.

Today we only showed a few examples of ex commands that work well with global,
we'll do some more complex examples next time.
