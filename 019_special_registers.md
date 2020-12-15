# Special registers

There are special registers that are intertwined with actions performed in vim.

Often performing an action (like a search, or inserting some text) will automatically populate these registers.

Here's a few:

- `.` - the last text inserted
- `:` - the last ex command
- `0` - the last yank
- `+` - the system clipboard
- `/` - the last search

# Exercises

## Exercise 1 - the dot register

Each time you:

- go from normal mode to into insert mode
- insert some text
- return to normal mode 

that text you inserted gets automatically put into the dot register.

We'll use this knowledge to backup our last insert.

- move your cursor to the colon at the end of the first line
- do `i` to enter insert mode and type `Boban whose name is lengthy and verbose<escape>`
- do `echo @.` (you should see the text you just typed)
- that's quite a lengthy title and we want to back it up into register `b`
- do `let @b = @.`
- check it worked by doing `echo @b`
- press `o` to start inserting on the next line
- type `And the people yelled: Yes he is <enter>`
- it's lucky we backed up Boban's title to register `b` because that last insert wrote over the `.` register
- confirm this with `echo @.`
- make sure your cursor is at the end of the second line (it should already be there)
- paste from the `b` register onto the end of the line using: `"bp`

```
And his name was: 
```

So the `.` register gives you a way to access lengthy bits of text so that you can save them for later reuse.

## Exercise 2 - the last samurai, I mean ex command

Each time you run an ex command with `:`, that gets put into the `:` register.

Let's run an ex command and then document what we did:

- do `:let @a = 'filling my register`
- move your cursor to the end of the line: "My last ex command was: "
- append the ex command by doing `":p`

```
My last ex command was: let @a = 'filling my register'
```

## Exercise 3 - the last yank

It's often the case that you're wanting to replace some text.

You've yanked the text you want to copy, and then you need to delete the old text you want to replace.

This doesn't work the way you'd expect though...

Extract the constant string "Warrior Boban" into a value:

- put your cursor somewhere on the first line of the block
- press `O` to insert a new line above
- type `val father = "Warrior Boban"<enter><escape>`
- put your cursor on "father" and yank it with `yiw` (yank inside little word)
- (we're copying it in preparation to paste it later in the spots where we'll extract it)
- move your cursor to the first "Warrior Boban" and delete it with `da"` (delete around quotes)
- do `P` to paste in "father" in its place
- what!?!? What happened - it pasted back in the text we just deleted - :shocked-parrot: 
    - yank, delete and paste are all using the same mysterious register (called the anonymous register)
- press `u` to undo
- do `"0P` to paste from the `0` register
- move your cursor to the next "Warrior Boban"
- do `da"` to delete it
- do `"0P` to paste from the `0` register again

```scala
val father = "Warrior Boban"

println("They called him " + "Warrior Boban")

val son = "Warrior Boban" + " junior"

println("He had a son called " + son)
```

People find it surprising that vim uses the same register for deleting and yanking and it often causes frustration.

This is because the editors they're used to don't put deleted text onto the clipboard - there's a separate "cut" operator for that.

However once you've gotten used to vim's mental model and the general form `"[REGISTER][OPERATOR][TEXT OBJECT]`,
it would be more confusing and inconsistent to make `y` and `d` (and `c`) work differently with regards to one
particular register. 

Instead learn to use the `0` register and embrace your inner vim.

## Exercise 4 - the system clipboard register

All of vim's registers are internal, but sometimes you want to copy text to the system clipboard.

_Usually_ this is represented by the `+` register.

Let's play with it:

- jump into some other app and copy something onto the system clipboard
- do `echo @+` to see what we got
- do `"+P` to paste it in somewhere
- put your cursor on this paragraph and do `"+yip` (yank this paragraph intro the `+` register)
- jump into some other app and see what's on your clipboard

Historically integration with the system clipboard has been notoriously troublesome with standard vim.
It can depend on how your vim was compiled and other things related to your OS.
So far I haven't hit any issues with neovim.

If `+` isn't working for you, also try the above with `*`.

## Exercise 5 (optional) - the last search register

Whenever you run a buffer search with `/`, that search term is automatically put into the `/` register.

This can be handy for seeing your last search and even modifying it from normal mode.

One odd quirk though is that you can't directly yank into the search register, but we can use `let @/ = ...`

We'll use this to our advantage to find Boban the great:

- run a case insensitive search for "Boban the git" by doing: `/\v\c<Boban the git><enter>`
- whoops, we just realized that we stuffed up our search - it should have been "Boban the great"
- let's see what we searched: `echo @/`
- paste that search term into the buffer somewhere: `"/P`
- fix it by changing "git" to "great", put your cursor on "git" and do `ciwgreat<escape>` which is:
    - `c` change
    - `iw` in little word
    - (puts you into insert mode)
    - `great`
- we can't copy directly into the search register, but we can copy into another temporary register `t`
- go to the start of the line your search term is on with `0`
- copy the current line into the `t` register with `"tyf>` which is:
    - `"t` - register t
    - `y` - yank
    - `f>` - until the closing caret
- set the search register from the `t` register: `let @/ = @t`
- press `n` to check if it worked

```
Anaboban the git
Boban the great
Boban the gitter user
```

Note that above was very convoluted and there are much much better ways to achieve this.
The main use for this would be documenting your search terms or binding shortcuts to them.

In particular if you want to fix previous searches, open the search with `/` then press `<c-f>` and you can see
all your search terms in a little buffer where you can edit them directly with the full power of vim's modal editing.

You might be wondering why we didn't do `"tY` to copy the search into register `t`.
Here `Y` means "yank current line".
Try this out and then do `let @/ = @t`. You'll find it doesn't match anything.
This is because you captured a sneaky newline character when you copied it.
If you do `echo @/` you'll see it on the end.
This is because `Y` is a line-wise copy, not a character-wise copy.

# Summary

Vim's got a few registers that are wired into standard editing tasks.

Learning these allows you to leverage vimscript and other clever tricks to save a lot of time.
