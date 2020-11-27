# Operation shortcuts

We've been heavily using the "operator + text object" syntax to make changes to our code.

Vim has a shortcut where typing the operation twice or using an upper cased version
applies the operator on a default text object (usually related to the current line).

For example:

- `yy` and `Y` yank the current line
- `D` deletes from the cursor to the end of the line (so part of a line)
- `dd` deletes the entire line
- `C` mirrors `D` in that it deletes to the end of the line then puts you in insert mode
- `cc` kinda mirrors `dd` in that it clears the line (doesn't delete it entirely) then puts you in insert mode
- `!!` will use the current line as input to the external program (and then you enter your program)
- `gUgU` will uppercase the whole line (`gUU` is a shortcut)
- `gugu` and `guu` are the lowercase equivalent

These are handy particularly because vim doesn't come with a simple text object for the current line (which is odd).
You have to resort to motions, highlighting or using the shortcuts above.

There is a nice plugin `kana/vim-textobj-line` which adds the `il` (inside line) and `al` (around line) text objects.

# Exercises

## Exercise 1

Remember this guy? It's a one line input so we can use `!!` here to prettify it.

- move your cursor to anywhere on the json line
- do `!!jq .<enter>`

```json
{"fire": ["joban", "joey-jo-jo-junior-shabadoo"], "dontFire": ["zij", "james", "jonathan"]}
```

## Exercise 2

Duplicate each of the lines below.

- move your cursor to anywhere on "Line 1" 
- do `Yp` (yank the current line then paste it below)
- move your cursor to anywhere on "Line 2"
- do `yyp` (yank the current line then paste it below)
- move your cursor to anywhere on "Line 3"
- do `.`
- (whoops! That pasted another copy of line 2 as "paste 'Line 2'" was our last change)
- do `u`
- do `Yp` or `yyp` again

```
Line 1
Line 2
Line 3
```

## Exercise 3

Completely delete all the lines that start with "Boban" and crop the lines with "Boban" in the middle by removing from "Boban" to the right.

- move your cursor to "Boban is supreme"
- do `dd`
- (now your cursor should be on the next line)
- move your cursor to the space just before "Boban"
- do `D`
- move your cursor to the next line down
- do `dd`
- (now your cursor should be on the last line)
- move your cursor to the space just before "Boban"
- do `D`

```
Boban is supreme
We ate a super supreme. Boban did too.
Boban is python
We saw a python eat someone. Boban is gone.
```
