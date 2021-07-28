# Visual mode modes

Last kata we introduced visual mode which we initiated by pressing `v`.

Turns out this is "character" visual mode.

There is also "line" and "block" mode.

# Shortcuts

`V` (capital v) puts you into line visual mode. The smallest unit of text that can be highlighted becomes a line.
This is much more convenient for line based operations (although we'll see that usually existing non-visual tools
replace most of the use cases)

`<c-v>` (control v) puts you into "block" mode which is a more unusual mode you won't have in many editors.

You can switch visual modes mid-highlight using the same shortcuts to enter those modes. For example:

- if you're already in line mode, pressing `v` will switch you to character mode
- if you're already in character mode, pressing `<c-v>` will switch you to block mode

# Exercises

## Exercise 1 - line mode

Move all the zero lines to below the one lines:

- move your cursor to anywhere on the first line of zeroes
- do `V` to start visual line mode
- press `jj` to go down two lines
- press `d` to delete (and cut) the lines
- move your cursor to the last line of ones
- press `p` to paste the lines

```
0000000000000000000
000000
000000000000
1111111111111111111
111111
111111111111
```

(Note this is just to demonstrate using visual line mode - a much better way to cut the 3 lines of zeroes
would have been do put your cursor on the first line of zeroes and do `d2j`, ie. "delete down 2 lines",
or `3dd`, ie. delete the current line 3 times)

Resist the urge to rely on visual line mode when there are faster, more vimmy ways to do things.

## Exercise 2 - switching modes

We'll use the snippet above to practice switching modes:

- put your cursor somewhere on the bottom line of zeroes
- do `v` to go into visual character mode
- press `k` a few times to extend the selection upwards
- (we work in a startup and requirements often change suddenly - a customer urgently needs us to switch to visual line mode)
- press `V` (capital v) to change our selection to line mode
- (turns out the customer was just a trouble maker - we're going back to the original requirements)
- press `v` to return to visual character mode
- press `v` or `<escape>` to return to normal mode

## Exercise 3 - building blocks

Slide our block of X's to the right side of the big block, then change the X's to Y's, then slide it to the left side:

- move your cursor to the top left 'X'
- do `<c-v>` to go into visual block mode
- move your cursor to the bottom right 'X' (note how your selection is a rectangle, it doesn't wrap around lines)
- do `d` to cut that block (this should cause the middle section of the outer block to shrink leaving a hole on the right)
- move your cursor to the end of the top line the hole is on
- do `p` to paste after the cursor
- move your cursor to the top left 'X'
- do `v` to enter character visual mode
- move your cursor to the bottom right 'X'
- (whoops! We're in character visual mode, not block mode!)
- do `<c-v>` to switch to block mode
- do `rY` to replace all the X's with Y's
- do `gv` to rehighlight the box of Y's
- do `d` to delete/cut the box
- move your cursor to the front of the top line missing a box
- do `p` to paste
- (whoops! That pasted it into the second column, we wanted to first)
- do `u` to undo
- do `P` (capital p) to paste _on_ the cursor, not _after_ the cursor


```
-------------------------
-------------------------
-----XXXXXXXXXXXXX-------
-----XXXXXXXXXXXXX-------
-----XXXXXXXXXXXXX-------
-----XXXXXXXXXXXXX-------
-----XXXXXXXXXXXXX-------
-------------------------
-------------------------
```

# Summary

There are three visual mode modes which make sense depending on the "shape" of text you're attacking.

It's easy to switch between them too if you make a mistake with your initial choice of shape.

Remember though that an experienced vimmer who is proficient with operations and text objects will not need visual mode that much.
So if you find yourself relying on it a lot, chances are you're using it as a crutch and it's hindering you from achieving vimmy transcendence.
