# Line and buffer text objects

Vim oddly doesn't come with built in text objects for the current line or buffer,
even though these are such common text objects.

Usually the current line is referenced implicitly by uppercasing or doubling an operator
(e.g. `dd` for delete current line, `Y` for yanking the current line).

For applying buffer wide changes you'd usually have to resort to an equivalent ex command which uses `%`
as the range, or visually selecting the entire buffer with `ggVG` then applying the operator.

For example to uppercase the entire buffer you could use `ggVGgU` or `:% normal gUU`.

# Plugins

As mentioned in the previous kata, we're using custom text objects provided by plugins.

For today you'll need:

- [vim-textobj-user](https://github.com/kana/vim-textobj-user)
- [vim-textobj-entire](https://github.com/kana/vim-textobj-entire)
- [vim-textobj-line](https://github.com/kana/vim-textobj-line)

The first plugin is a general purpose tool for building custom text objects.

The latter two use it to define buffer and line text objects.

## "Entire"

The mnemonic for the buffer is 'e' for "entire".

('b' is already being used used for brackets, e.g. `di)` is equivalent to `dib`)

The plugin provides two text objects:

- `ae` - the entire buffer
- `ie` - the entire buffer minus whitespace at the top and bottom

This means we can uppercase the entire buffer with `gUae` or format the buffer with `=ae`
which is a fair bit cleaner than having to use a visual selection.

We also get an `ie` text object as a bonus which would be fiddly to replicate using visual line mode.

## "Line"

The mnemonic for the line is 'l'.

The plugin provides two text objects:

- `al` - the entire line (but not the newline character)
- `il` - the line minus any leading or trailing whitespace

```
   def foo(): Unit = {     <-- Some trailing whitespace
--------------------------- al
   -------------------      il
```

Whilst there are built in line based operations, sometimes it's still nice to have an explicit text object.
That way you can use a smaller more consistent mental model of "operator + text object" with less ad-hoc
commands to remember.

Likewise there might be times where you want to use the "inside line" to get a trimmed result,
e.g. if you're copy-pasting a line of text out.

Even if you want the whole line, yanking with `Y` or `yy` capture that new line character at the end of the line,
and vim interprets the register as being a line based yank rather than a character based yank.
That means when you paste it, it will paste it below the current line not after your cursor on the same line.
In the cases where you want it after your cursor this is annoying and can be confusing.

Another place where those trailing newlines are annoying is for copy-pasting text out of vim into other apps,
e.g. something expecting a password, a url field etc...
That extra subtle newline character often doesn't play well with those apps and cause subtle issues.

Note that if you _don't_ have a text object for the current line, you can get rid of this pesky newline
by using the motion `g_` which takes you to the end of your line just before the newline.
For example `0"+yg_` copies the line onto the clipboard:

- `0` - go to the start of the line
- `"+y` - yank operator - what comes next goes into the `+` register (system clipboard)
- `g_` - motion to go to the end of the line (which becomes a text object for all the characters up to there)

# Exercises

## Exercise 1 - buffer operations

We'll do some buffer level operations, but not in this buffer,
otherwise we won't be able to follow the instructions!

- do `:new` to open a new empty buffer in a split
- read this buffer in by doing `:read 036_line_and_buffer_text_objects.md`
    - (notice how it added an extra blank line at the top of the buffer,
       this is because `read` always dumps _below_ the line)
- do `vie` (visually select insides the buffer)
- use `o` to jump between the top and bottom anchor
- (notice how the blank line at the top of the file is _not_ selected)
- do `v` to leave visual selection
- do `gUae` to uppercase the entire buffer
- do `u` to undo
- do `=ae` to format the entire buffer (results may differ!)
- do `u` to undo
- do `>ae` to right shift the entire buffer
- do `u` to undo
- do `vaeJ` to highlight the entire buffer then join it into a single line
- do `:q!` to discard the buffer and close the split

## Exercise 2 - line operations

We'll play with the line of scala code below comparing our text object to the built in line based operators:

- put your cursor on the line
- do `val` to convince yourself there is leading and trailing whitespace, then `v` to return to normal mode
- do `vil` to convince yourself that the `il` text object ignores trailing whitespace, then `v` to return to normal mode
- do `gUil` to uppercase the line, then `u` to undo
- do `>il` to right shift the current line, then `<al` to reverse it (inside or around doesn't matter here)
- do `dil` to delete inside the line, then `u` to undo
- do `"tyal` which yanks the line into register `t` (for "text object")
- do `"dY` which yanks the whole line using regular `Y` into the register `d` (for "default"), then do `:echo @d`
- do `:echo @t` and `:echo @d` and note the extra blank line for `@d`
- to confirm they're different do `:echo @t == @d` (should get `0` printed for false)
  - do `:echo stridx(@d, @t)` to show that `@t` is a substring of `@d` at index 0
  - do `:echo stridx(@t, @d)` to show that `@d` isn't a substring of `@t`
- do `"iyil` to yank inside the line into register `i` (for "inside")
- do `:echo stridx(@t, @i)` to show that `@i` is a substring of `@t` at index 2

```scala
  def foo(): Unit = {     
```

# Conclusion

These text objects fill a hole in vim that allows us to use a more unified approach to editing.

The line based text object is particularly useful for avoiding the trailing newline charater.
