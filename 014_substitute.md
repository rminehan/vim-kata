# Substitute

Substitute is vim's built-in find-replace mechanism.

The general form is:

```
:[RANGE] substitute /[FIND PATTERN]/[REPLACE PATTERN]/[FLAGS]
```

For example to replace "boban" with "enxhell" on lines 1-5 you'd do:

```
:1-5 substitute /boban/enxhell/g
```

# Quick range recap

Here's some example ranges to jog your memory:

- `%` - entire buffer
- `5` - just line 5
- `.` - just the current line (line the cursor is on)
- `5,15` - lines 5 to 15 inclusive
- `.,20` - the current line until line 20
- `.,.+5` - the line the cursor is on to 5 lines after the cursor
- `'<,'>` - the previously visually selected lines

# Lazy vimmers

Vimmers don't want to write "substitute" repeatedly, so vim lets you just write "s".

Likewise you don't have to have a space before and after "substitute".

So the example can be shortened:

```
:1-5 substitute /boban/enxhell/g

:1-5s/boban/enxhell/g
```

# Visual mode

Recall from a previous kata, that if you visually select a bunch of lines then press `:`,
it will prepopulate the range with `'<,'>` which effectively represents the lines that were highlighted.

- `'<` means "the start line of the previous visual selection"
- `'>` means "the last line of the previous visual selection"

Hence `'<,'>` is just sticking a comma between them to define a range.

You could do something like `'<,20` to represnt from the start of the last visual selection to line 20.

# Replace preview

Neovim has a nice interactive preview which shows you the changes as they occur.

# Common regex args

- `i` - case insensitive
- `g` - all matches per line (not just the first)
- `c` - interactive replace

# Exercises

## Exercise 1

Replace all the Boban's with Enxhell's below:

- put your cursor on the first line of text
- do `:.,.+2s/Boban/Enxhell/<enter>`, breaking that down:
    - `:` - enter command mode
    - `.,.+2` - the current line and down 2 lines
    - `s` - short for "substitute"
    - `/Boban` - the search term
    - `/Enxhell` - the replace term
    - `/` - flags (none in this case
    - `<enter>` - lock it in Eddy

```
Boban is a plucky young Albanian.
Boban would often say "mate" to sound Australian. That's very Boban!
Have you met Boban?
```

Whoops! We missed the second Boban on the middle line.

Remember that by default substitute will only replace the first match it encounters on each line.

We need to add `g` for "global" to our args.
In my experience this is almost always what you want, so much so that it's just programmed into my muscle memory to always add a `g`.

Hit `u` to undo, then repeat the above but add `g` after the final `/`.

## Exercise 2

Replace all Boban's with Enxhell's in the block below in a case insensitive way.

Note as well we don't want to replace Bobanita, she's fine.

Note as well that Boban's friends call him "Bobn" as a nickname.

- move your cursor to the top line of text
- do `:.,+3s/\c\v<boba?n>/Enxhell/g<enter>` which breaks down to:
    - `:` - enter command mode
    - `.,+3` - the current line down 3 lines
    - `s` - substitute
    - `/\c\v<boba?n>` - let's break this down even more:
        - `/` - start the search pattern
        - `\c` - case insensitive (we want to match "Boban", "boban", "BOBAN" etc...)
        - `\v` - very magic mode to ensure subsequent special characters are treated as regex characters
        - `<` - left word boundary, to exclude names like "haboban"
        - `boba?n` - match "boban" and "bobn" (ie. the 'a' is optional)
        - `>` - right word boundary, to exclude names like "bobanita"
    - `/Enxhell` - the replace term
    - `/g` - replace all matches on each line
    - `<enter>` - fire the missiles

```
Boban is a plucky young Albanian, not like Bobanita.
She'd say: "That boban, he's a real card".
Boban would often say "mate" to sound Australian. That's so bobn!
Have you met Boban or Bobanita?
```

# Relative ranges

The observant Jonathan might have noticed that for the range we used:

```
.,+3  instead of
.,.+3
```

This is a little shortcut: `+X` and `-X` are assumed to be relative to the current line if you don't put in the line number. 

# Junior?

In exercise 2 we mapped all searches to "Enxhell" which meant:

- "boban" -> "Enxhell"
- "bobn" -> "Enxhell"

It's quite a simplistic replacement that throws away the capitalization of the original text as well as the whether the original
text was being used as the formal "Boban" or the nickname "Bobn".

It would be nice for example to map "bobn" to Enxell's nickname "junior" to preserve the original nuance of the text.

Likewise sometimes the plural form of a word change. For example the plural of "Boban" is "Bobans"
but the plural of "Enxhell" is "Enxhellens".
If we did a direct replacement of Boban to Enxhell, then "Bobans" would become "Enxhells" which is incorrect.

So this section is really saying that there are times we need more sophisticated replace logic.

This is beyond today's kata, but you can look up:

- using back references (e.g. `\1` syntax)
- using vimscript in your replacement
- Tim Pope's awesome abolish plugin which contains the `Subvert` command (see [kata 52](052_subvert.md))

# Search + dot?

Do we really need the substitute command? Often the answer is no.

In exercise 1 we wanted to do a vanilla replacement of "Boban" to "Enxhell". We could have:

- search for Boban with `/Boban`
- hit `n` until you're on the first Boban in the text block
- do `ciwEnxhell` to change it to "Enxhell"
- hit `n` to move to the next match
- do `.` to repeat the last action
- continue on this way

```
Boban is a plucky young Albanian.
Boban would often say "mate" to sound Australian. That's very Boban!
Have you met Boban?
```

This is equivalent to the interactive substitute mode, because at each point you get to choose what to do.

If you wanted to upper case all the Boban's you could do a similar pattern to the above doing:
`gUiw` on the first, then just use `.`.
This avoids building complex find-replace patterns.

Substitute will still be useful in some cases (e.g. if there's many many replacements or you're writing a script),
but you don't need it as much as you would in other editors because of the very powerful "operator + text object" combined with dot.

It's because of this that I've left the substitute until later to encourage using the very general powerful tools
from the earlier lessons. There was a simialar rationale for delaying visual mode and macros.
