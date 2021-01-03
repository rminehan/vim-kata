# Advanced regex 3 - programmatic regexes

In today's kata we're going to programmatically build regex patterns using vimscript.

# Quick vimscript recap

Vimscript is the language your config file is written in and it's the language you use when you enter `:` commands.

It's a fully fledged language in the sense that it has types like string and numeric, and operators and libraries
to use with those types.

Recall that `.` is the string concatenation operator, e.g. `:echo 'abc' . 'def'` should print 'abcdef'.

Also remember that our registers we record macros into are just variables in vimscript we can programmatically modify.
e.g. `:let @q = 'abc'` is the same as doing `qqabcq` from normal mode.

# execute

Today we'll look at the `execute` function which is a vimscript function you pass a string to,
and it executes it as if you typed it as an ex command using `:`.

For example:

```vim
" Set line wrapping
:execute 'set wrap'`

" Inspect the current setting
:execute 'set wrap?'
" Should print 'wrap'
```

This isn't a very interesting example as you could have just typed `:set wrap` directly.

`execute` will become useful though when we need to mix dynamic and static concepts together to build complex and
configurable patterns.

# Escaping backslash

Like bash, in vimscript the backslach character is understood to be an escape character for double quoted strings,
but it's literal in single quoted strings:

```vim
let i = "\\abc"

" Should print \abc
echo i

let j = '\\abc'

" Should print \\abc
echo j
```

So if we use double quoted strings to build our regexes in the exercises, we'll need to escape the backslashes,
e.g. very magic would be `"\\v"`.
This makes very magic mode even more useful for avoiding a lot of backslash noise.

Generally we'll use single quoted strings to avoid this, but if a regex contained a single quote we'll need to change tactics.

Things get more complex as well when you're executing commands within execute that themselves take strings.
For example executing `echo '\'`. In this case we can change what's being echo'd to use double quotes,
and use single quotes to wrap the command as a whole:

```vim
execute 'echo "\\"'
```

As the logic gets more increasingly nested, escaping can start to get a bit mind bending...

# Exercises

As usual read through the setup rules from this section in the first advanced regex kata.

## Exercise 1 - execute simple search

It turns out that you can run searches from command mode.

- put your cursor on "abc" in the block below
- do `:/abc<enter>`

```
abc   acdc
ABC   aBc
```

This is just the same as directly running `/abc` from normal mode (except it's not interactive regarding `incsearch`).

This means we can use `execute` to run searches (as `execute` always runs from command mode).

- again put your cursor on "abc" in the block above
- do `:execute '/abc'<enter>`
- do `:echo @/` to see your last executed search (should be "abc")

You should get the same matches as before.

## Exercise 2 - introducing dynamic logic

Vimscript has a function `strftime` that takes a format string and produces date time info.

Do `:echo strftime('%Y')<enter>`. It should print the current year.

Let's use vimscript's concatenation operator `.` to combine this with a static search to find references to
the current year in our buffer.

Hit `:` then type in:

```vim
execute '/YEAR:' . strftime('%Y')<enter>
```

Assuming the current year is something from 2021-2026, then this should match something in the block below:

```
YEAR:2021  YEAR:2024
YEAR:2022  YEAR:2025
YEAR:2023  YEAR:2026
```

Do `:echo @/` to see what search was performed. It should be something like "YEAR:2021".

## Exercise 3 - incorporating the current line

Vimscript has a function `line` which takes a string expression and tells you the line number as an int.

For example:

- `line('.')` returns the current line the cursor is on
- `line('$')` returns the line number of the last line in the buffer
- `line('w0')` returns the line number of the first line visible on screen

Do `:echo line('.')<enter>` to try it out. See `:help line` for more info.

Put your cursor some from line 140-155 and run:

```vim
:execute '/LINE:' . line('.')<enter>
```

```
LINE:140    LINE:144    LINE:148    LINE:152
LINE:141    LINE:145    LINE:149    LINE:153
LINE:142    LINE:146    LINE:150    LINE:154
LINE:143    LINE:147    LINE:151    LINE:155
```

Do `:echo @/` to confirm the last search (will be something like `LINE:141`)

## Exercise 4 - incorporating registers

Remember that registers can be accessed from vimscript using the `@` sign and their name.
We'll create a search pattern that searches for whatever we last yanked with word boundaries.

Remember that when you yank something without specifying a register,
it goes into the default/anonymous register `"`.

Yank "Boban" into the default register by either:

- putting your cursor between the quotes and doing `yiw` or `yi"`
- running `:let @" = 'Boban'<enter>`

Confirm it's contents with `:echo @"<enter>`.

Run this:

```vim
:execute '/\v<' . @" . '>'
```

It searches for `\v<Boban>` (ie. "Boban" with word boundaries):

```
Bobanita     Ahboban    Boban     boban         Boban_smith        Frodobanita
McBoban      Biban      Frodoban  boban-jones   Bobanita.Hayworth
```

Now we'll rerun it with a different value yanked:

- put your cursor on this "Frodoban"
- yank with `yi"`
- hit `:`
- use `<c-p>` to dial up the previous `execute ...` command
- hit enter to run it again
- confirm the pattern searched with `:echo @/<enter>`

Now we'll show using `@:` to rerun the previous ex command:

- yank this: `\cB.ban` by putting your cursor between the backticks and doing `yi<backtick>`
- hit `@:`

This should match the usual suspects and also "Biban".

## Exercise 5

For this exercise we'll devise a way to search for different first and last name combinations.
We'll get to a point where we can just update specific registers and then rerun a complex search.

We'll put first names into register `f` (for "first") and last names into register `l` (for "last").
Then we'll construct a search describing:

```
first name + delim + last name

where delims can be: space, underscore or hyphen.
```

Our pattern will also read case sensitivity settings from register `c`.
Expected values are either `\c`, `\C` or just empty.
It will get inserted near the start of our pattern but it could go anywhere.

We're also going to use `%V` from our first advanced regex kata to restrict the matching to
the most recently selected area.
Also remember the odd glitch where `%V` causes case insensitivity to turn off.
This has the effect that if the `c` register is not set, the match will be case sensitive.

- visually select the block below then hit escape
- do `:let @c = '\c'<enter>`
- do `:let @f = 'boban'<enter>`
- do `:let @l = 'jones'<enter>`
- do `:execute '/\v%V' . @c . @f . '[ _-]' . @l<enter>`
- do `:echo @/<enter>` to see the final search performed

```
Bobanita Smith    Ahboban        Boban         boban jones
McBoban jones     Biban          Snoban        boban-jonesmouth
bobanita_jones    BOBAN jones    Boban Jones   Ahboban_smith
```

That should match:

- "boban jones"
- "(Mc)Boban jones"
- "boban-jonesmouth".
- "Boban Jones"
- "BOBAN jones"

Next we'll turn off the case insensitivity flag and make the first name title case insensitive and not change the last name.

- select the block below as usual
- do `:let @c = '\C'<enter>`
- do `:let @f = '[Bb]oban'<enter>`
- do `:` then hammer `<c-p>` until you've dialed up the last `execute ...` command, then hit enter

```
Bobanita Smith    Ahboban        Boban         boban jones
McBoban jones     Biban          Snoban        boban-jonesmouth
bobanita_jones    BOBAN jones    Boban Jones   Ahboban_smith
```

## Exercise 6 - automating the search

Carrying on from exercise 5, we want to be able to quickly update our search terms then run a new search.

Hitting `:` and hammering `<c-p>` is a bit tedious.

We'll map question mark to run the execute command. Hit `:` then run this:

```vim
nnoremap ? :execute '/\v%V' . @c . @f . '[ _-]' . @l<enter>
"          ------------------------------------------------ --------
"           When you hit `?` from normal mode, it's as if   Then press
"                           you typed these keys            enter to submit this
```

Note the `<enter>` you type there is the actual string `<enter>` (and you'd press enter _after_ it to lock in the command).
Because it's in carets, vim interprets it to mean the enter key. We could also have put in .

To clarify, we're describing literal key presses for vim to enter when we hit `?`.
So we start typing `:execute .... @l` and then we need to hit enter to submit the execute command.

We're all setup for some fast searching now:

- visually select our comprehensive database of Enxhell's cousins below like usual 
- do `?` (that should bring up similar matches to exercise 5)
- do `:let @c = '\c'<enter>` then hit `?` to run a new search
- do `:let @f = 'Bobanita'<enter>` then hit `?`
- do `:let @f = '(mc|ah)@<=boban'<enter>` then hit `?`

```
Bobanita Smith    Ahboban        Boban         boban jones
McBoban jones     Biban          Ahboban smith boban-jonesmouth
bobanita_jones    BOBAN jones    Boban Jones   Ahboban_smith
Bobanita Bobans   Ahjoban Jones  Boban Smith   Anaboban.smith
```

As a list little improvement we'll make the delims configurable too:

```vim
nnoremap ? :execute '/\v%V' . @c . @f . '[' . @d . ']' . @l<enter>
```

- do `:let @d = ' .-_'<enter>` (space, period, hyphen and underscore)
    - note that `.` in a character class means literal '.'
- do `?` to rerun the search with the updated regex
- do `:let @f = 'ah\S*'<enter>` ("ah" followed by 0 or more non-whitespace characters)
- do `?`
- do `:let @l = 'smith'<enter>`
- do `?`

# Conclusion

We can use the `execute` command to compose regexes on the fly programmatically from editor state
like registers and vimscript functions.

This lets us build more powerful plugins and scripts.

# Further reading

Here's a complex `execute` example of
[updating copyright info for the current year](https://vim.fandom.com/wiki/Automatically_Update_Copyright_Notice_in_Files)
