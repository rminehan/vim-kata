# Marks

Marks are vim's version of bookmarks.

You can place a mark and then come back to it later.

Like with registers, each letter of the alphabet is a slot for holding a mark.

For example you might mark the current line with `x` (x marks the spot!) to come back to it later.

One difference to registers though is that there are also upper case marks.

# Shortcuts

There's 3 short cuts to remember:

(1) Creating a mark: `m[LETTER]`, e.g. `mx`.

(Just remember "m" for "mark")

(2) Jump to a mark's line: `'[LETTER]`, e.g. `'x`.

(Simply remember "apostrophe" for 'jumping to the line my mark is on') (I wrapped it in apostrophes)

(3) Jump to a mark's character: `<backtick>[LETTER]` e.g. `<backtick>x` (I can't escape backticks)

(Remember "backtick" for `jumping to the character my mark is on`)

# Buffer local vs Global marks

Lower case marks are "buffer local", ie. scoped to the buffer they're in.

So you can have the mark `x` in many buffers.

When you're in a particular buffer and do `'x` to jump to mark `x`, it will see if there's one for your buffer,
it won't jump into other buffers.

Upper case marks however are "global". That means there is only one of each.

You can jump to a global mark from anywhere.

# Exercises

## Exercise 1 - buffer local marks

- put your cursor on the "1" in the heading for this exercise ("Exercise 1 - ...")
- do `mx` to place a mark
- do `5j` to go down 5 lines
- do `'x` to jump back to the line of your mark
- (this will put your cursor at the start of the line)
- do `5k` to go up 5 lines
- do `<backtick>x` to jump back to the "1" in the heading

## Exercise 2 - duplicate buffer local marks

- put your cursor somewhere on this line and mark it with `mx` to mark these instructions
- do `:split` to split this window
- do `:edit 026<tab><enter>` to bring up the previous kata
- do `'x` to try to jump back to mark `x` in this buffer
- (it should fail as it was looking for a mark `x` in its own buffer)
- do `mx` to place a mark whereever you are in that buffer
- (now we have two `x` marks - each in their own buffer)
- move your cursor off that line
- do `'x` to return to that line (just showing it works)
- do `:q` to exit that split
- do `'x` to return to your mark in this buffer

## Exercise 3 - global marks

- put your cursor on this line and do `mX` to mark it with a global mark
- do `:split 026<tab><enter>` to bring up the previous kata in a split
- do `'X` to jump back to this buffer
- do `<c-o>` to jump back to the other buffer
- do `mX` (this will move the mark to the other buffer)
- do `<c-i>` to jump back to this buffer
- do `'X` to jump back to the previous buffer
- do `:q` to exit that split

Note the above sneakily introduced the `<c-o>` and `<c-i>` operators for traversing the "jump list".

# Are marks useful?

Early in my vim life I used these custom marks a lot, but I've found as time goes on I only need them in
a handful of situations, particularly as there are other really fast mechanisms for finding text and moving between buffers.

That is why they've been pushed so far back in the kata - they're important to know but not as useful as other concepts.

The built-in marks we'll cover in next lesson _are_ quite useful though.

# If you use marks a lot...

If you do end up using custom marks a lot, have a look at the [ShowMarks](https://github.com/vim-scripts/ShowMarks)
plugin which will visually show your marks in the left margin of your editor.

# Summary

Marks can be a useful way to mark your place and jump back to it later.

You can use internal mnemonics (like `p` for "ProxyService") to help you remember which mark is for which.

Next kata we'll cover some really useful built in marks.
