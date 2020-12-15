# Recording macros

Hopefully you're now pretty comfortable with the idea of registers being little buffers or variables that contain text.

So far we've always _interpreted_ those buffers as text that we would paste into a document.

There is however another way to interpret those characters, and it's as a set of vim instructions.
Code is data? Data is code? What's going on?

# Exercises

## Exercise 1 - playing macros

We're going to write some instructions into a register and then "play" them.

- do `:let @a = '0xxxx'<enter>`
- move your cursor to line 1 of the block below
- do `@a` ("play" register `a`)
- you should see the first 4 characters disappear because we literally did:
    - `0` - move to start of line
    - `xxxx` - delete the character under the cursor 4 times (could have also done `4x`)
- move your cursor to the next line and do `@a` again
- move your cursor to the final line and do `@@`
    - this means "play the last played macro"
    - it's much faster to double tap the `@` key

```
001 Line 1
002 Line 2
003 Line 3
```

It's important to emphasize here that register `a` is still just some characters.

The new thing is that we're now _interpreting_ it as instructions, rather than "text" for a buffer.

Calling a register a "macro" just means: The intention for the data on this register is to be played as instructions.

You can still use all your tricks like:

- inspecting them, ie. `echo @[REGISTER]`
- programmatically building them, e.g. `let @a = @b . @c . 'foo'`
- pasting from registers into buffers, e.g. `"aP`
- copying from text objects into registers, e.g. `"ayiw`

## Exercise 2 - recording a macro

In exercise 1 we programmatically built our macro with `:let @a = '0xxxx'`.

What would be nice though would be to actually practice it on a sample and record what we did.

Vim lets you do this with "record" - `q[REGISTER]`.

- put your cursor somewhere on line 1
- start recording into register `a` by doing: `qa`
- do `04x`
- press `q` to stop recording
- inspect register `a` with `echo @a`
- move your cursor to line 2 and do `@a` to play the macro 
- move your cursor to line 3 and do `@@` to play the last played macro

```
001 Line 1
002 Line 2
003 Line 3
```

To sum that up, the nice thing about recording is that it lets you interactively build your macro.

You can enter your keystrokes into the buffer whilst seeing what effect they have on a sample piece of data.

Again though there's nothing stopping you programmatically building your macros.

## Exercise 3 - combining macros

From exercise 2, our `a` macro deletes the first 4 characters on a line.

We'll make another macro to upper case a line - then combine them into a super macro that does both.

- put your cursor somewhere on line 1
- do `qb` to start recording into register `b`
- do `gUU` to upper case all the text on the current line
- do `q` to stop recording
- confirm the contents of register `b` with `echo @b`
- (we'll combine them into a single macro that both deletes the first 4 characters and uppercases the remaining line)
- do `:let @c = @a . @b` (remember that `.` is string concatenation in vimscript)
- move your cursor to line 2 and do `@c` (it should now be "LINE 2")
- move your cursor to line 3 and do `@@` (it should now be "LINE 3"

```
001 Line 1
002 Line 2
003 Line 3
```

So if our macros compose nicely, we can build new ones quite easily.

## Exercise 3 - bulk macro application 

Usually if you've overcome your laziness and written a macro it's because there were too many lines for you to manually change.

You can use a count with macro application to apply it many times, e.g. `3@a` will apply the `a` macro 3 times.

There's 8 lines below that we could apply macro `c` to from exercise 3:

- put your cursor on line 1
- do `8@c`
- yikes what happened!? The line just disappeared - that's because we applied the macros 8 times to the same line
    - we need the macro to move us to the next line when it's done so we're ready for the next application
- do `:let @c = @c . 'j'<enter>`
    - we're effectively appending `j` to our macro which moves us down 1 line
- do `echo @c` to see what it looks like now, should be: "0xxxxgUUj"
- do `8@c` again (should work properly this time)

```
001 Line 1
002 Line 2
003 Line 3
004 Line 4
005 Line 5
006 Line 6
007 Line 7
008 Line 8
```

## Exercise 4 (bonus) - 800 Jonathan's standing on a wall

Imagine we needed to generate a big block of text like: 

```
[800] there were 800 Jonathan(s) standing on the wall, one fell down, now:
[799] there were 799 Jonathan(s) standing on the wall, one fell down, now:
...
[001] There were 1 Jonathan(s) standing on the wall, one fell down, now:
[000] There were 0 Jonathan(s) standing on the wall
```

We're not going to do that by hand! We're programmers!

We will write a macro which generates a new line off the previous line. Essentially it will:

- copy the previous line
- de-increment each number

We bootstrap it by creating the first line 1 manually, then we run that macro ~800 times using a count.
Then we can manually cleanup line 000 as it's a little different.

- move the cursor to the top line of backticks and press `o`
- type out `[800] there were 800 Jonathan(s) standing on the wall, one fell down, now:<escape>`
- do `qj` to record into the `j` register ('j' for jonathan)
- do `Y` to yank the current line
- do `p` to past it below (and move your cursor to it)
- do `0` to move your cursor to the start of the line (see notes about this below)
- do `<c-x>` to deincrement the first number on the line (that one in square brackets)
- do `l` to move your cursor one character to the right
- do `<c-x>` to deincrement the next number on the line
- do `q` to stop recording
- do `5@j` to play it 5 times (a sanity check)
- if all looks good, unleash the beast! We have 795 lines left to generate: do `794@@`
- (manually tidy up the last line)
- put your cursor on the 0 line
- do `f,` to move the comma
- do `D` to delete to the end of the line

A note about `0`: Generally when you build complex macros it's good practice to explicitly position yourself
when you do things like moving to a new line.
This avoids some subtle bugs and weird edge cases.
As you build macros yourself you'll start to realize a set of "best practices" or "macro hygene" that make them less buggy. 

# Summary

"Macros" are a reinterpretation of registers for use as instructions rather than text in a buffer.

Don't forget though that they are still just text in a register
and you can use all of your register foo to print them, edit them, combine them etc...

I deliberately left this section until after covering register basics to reinforce register fundamentals
as most people jump straight to macros and just think of registers as being macros.
Then they don't possess the skills to work cleverly with their macros using their register fundamentals.

That's it for registers now.
