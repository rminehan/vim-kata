# Advanced regex 3 - programmatic regexes

In today's kata we're going to programmatically build regex patterns using vimscript.

This is not something you'd do in regular coding, it's more for scripting and building powerful shortcuts.

# Quick vimscript recap

Vimscript is the language your config file is written in and it's the language you use when you enter `:` commands.

It's a fully fledged language in the sense that it has if statements, loops,
types like string and numeric and a standard library.

Recall that `.` is the string concatenation operator, e.g. `:echo 'abc' . 'def'` should print 'abcdef'.

Also remember that registers we record macros into are just variables in vimscript we can programmatically modify.
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

# Exercises

See the [setup guide](advanced_regex_exercises_setup.md).

## Exercise 1 - execute simple search

It turns out that you can run searches from command mode.

- do `:/abc<enter>`

```
abc   acdc
ABC   aBc
```

This is just the same as directly running `/abc` from normal mode (except it's not interactive regarding `incsearch`).

This means we can use `execute` to run searches (as `execute` always runs from command mode).

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

Assuming the current year is something from 2024-2029, then this should match something in the block below:

```
YEAR:2024  YEAR:2027
YEAR:2025  YEAR:2028
YEAR:2026  YEAR:2029
```

Do `:echo @/` to see what search was performed. It should be something like "YEAR:2021".

## Exercise 3 - incorporating the current line

Vimscript has a function `line` which takes a string expression and tells you the line number as an int.

For example:

- `line('.')` returns the current line number the cursor is on
- `line('$')` returns the line number of the last line in the buffer
- `line('w0')` returns the line number of the first line visible on screen

Do `:echo line('.')<enter>` to try it out. See `:help line` for more info.

Put your cursor somewhere from line 100-115, take note of the line number, then run:

```vim
:execute '/LINE:' . line('.')<enter>
```

```
LINE:100    LINE:104    LINE:108    LINE:112
LINE:101    LINE:105    LINE:109    LINE:113
LINE:102    LINE:106    LINE:110    LINE:114
LINE:103    LINE:107    LINE:111    LINE:115
```

Do `:echo @/` to confirm the last search (will be something like `LINE:101`)

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
- hit `:` and use `<c-p>` to bring up the execute command from above
- hit enter

This should match the usual suspects and also "Biban".

Note that this is quite similar to the `*` operator which searches for the word under the cursor
with word boundaries. You now have enough tools to build an operator like that yourself.

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
By convention it will get inserted near the start of our pattern but really it could go anywhere.

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
- "boban-jones(mouth)".
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

Now that it's case sensitive, we lose some matches.

We've built a basic search tool. We can update specific registers and rerun our search command.

## Exercise 6 - lookahead

For fun let's add a lookahead from the previous kata to only match the lastname "jones" if it has a letter after it:

- select the block below as usual
- do `:let @l = 'jones\a@='<enter>`
- if you have fzf.vim installed, do `:History:<enter>` and zoom in on the execute command, otherwise use `<c-p>` like before

```
Bobanita Smith    Ahboban        Boban         boban jones-mouth
McBoban jones     Biban          Snoban        boban-jonesmouth
bobanita_jones    BOBAN jones    Boban Jones   Ahboban_smith
```

Just the "boban-jones(mouth)" should match.

## Exercise 7 (optional) - automating the search

Suppose we want to be able to quickly update our search terms then run a new search.

Hitting `:` and hammering `<c-p>` is a bit tedious.

We'll map question mark to run the execute command. Hit `:` then run this:

```vim
nnoremap ? :execute '/\v%V' . @c . @f . '[ _-]' . @l<enter>
"          ------------------------------------------------ --------
"           When you hit `?` from normal mode, it's as if   Then press
"                           you typed these keys            enter to submit this
```

The mapping above is describing the literal key presses you want when you hit `?`.
The tricky part is how do you represent hitting enter as part of the inner `execute` command?
If you press enter, then vim takes that to mean you're finished with the `nnoremap`.

The solution is to write out the literal text `<enter>` or use the literal enter character .
Hence above you type `<enter>` to represent pressning enter for the `execute` command,
then you press the actual enter key to finish the `nnoremap` command.
Layers upon layers!

We're all setup for some fast searching now by just hitting `?`.
Below we'll keep updating our search terms then rerun `?` each time:

- visually select our comprehensive database of Enxhell's cousins below like usual then hit escape
- do `:let @l = 'jones'<enter>` then hit `?`
- do `:let @c = '\c'<enter>` then hit `?`
- do `:let @f = 'Bobanita'<enter>` then hit `?`
- do `:let @f = '(mc|ah)@<=boban'<enter>` then hit `?`

```
Bobanita Smith    Ahboban        Boban         boban jones
McBoban jones     Biban          Ahboban smith boban-jonesmouth
bobanita_jones    BOBAN jones    Boban Jones   Ahboban_smith
Bobanita Bobans   Ahjoban Jones  Boban Smith   Anaboban.smith
```

## Exercise 8 (optional) - changing the delims

So far our delims between first and last name have been hard-coded to only allow space, `_` and `-`.

For this exercise we'll make them configurable too by reading them from `@d`
and wrapping them in a character class `[ ... ]`.

Let's rebind `?` to a modified command:

```vim
nnoremap ? :execute '/\v%V' . @c . @f . '[' . @d . ']' . @l<enter>
```

(above it's implied you type out all those characters literally then hit enter)

- visually select the below as usual
- do `:let @d = ' .-_'<enter>` (space, period, hyphen and underscore)
    - note that `.` in a character class means literal '.'
- do `?` to rerun the search with the updated regex
- do `:let @f = 'ah\S*'<enter>` ("ah" followed by 0 or more non-whitespace characters)
- do `?`
- do `:let @l = 'smith'<enter>`
- do `?`

```
Bobanita Smith    Ahboban        Boban         boban jones
McBoban jones     Biban          Ahboban smith boban-jonesmouth
bobanita_jones    BOBAN jones    Boban Jones   Ahboban_smith
Bobanita Bobans   Ahjoban Jones  Boban Smith   Anaboban.smith
```

# Escaping backslash

In our exercises we've been using single quotes for our ex commands because it avoids a lot of backslash noise.

Like bash, vimscript supports double quoted and single quoted strings,
and like bash, they treat backslach characters differently.

```vim
let i = "\\abc"

" Should print \abc
echo i

let j = '\\abc'

" Should print \\abc
echo j
```

So with single quotes, a backslash is a literal backslash, but with double quotes, it's an escape character.
This means that it you want a literal backslash with double quotes, you need to escape it: `"\\"`.

Because our regexes contain a lot of backslashes already (e.g. `\v`, `\d`, `\w`, `\a`, `\S`),
it makes much more sense to use single quoted strings to avoid noisy double backslashes.

But! If a regex contained a literal single quote we might need to use double quotes.

Things get more complex when you're executing commands within commands.
For example executing `echo '\'` as part of an `execute` command.
Here we'd need to use both types of quotes:

```vim
execute 'echo "\\"'
```

As the logic gets more increasingly nested, escaping can start to get a bit mind bending...

# Conclusion

We can use the `execute` command to compose regexes on the fly programmatically from editor state
like registers and vimscript functions.

Realistically this isn't something you'd do in your day to day editing,
but it would allow you to start writing powerful search plugins or shortcuts.

# Further reading

Here's a complex `execute` example of
[updating copyright info for the current year](https://vim.fandom.com/wiki/Automatically_Update_Copyright_Notice_in_Files)
