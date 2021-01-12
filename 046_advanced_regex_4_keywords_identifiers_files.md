# Advanced regex 4 - keywords, identifiers and files

Programming languages typically have concepts of "keywords" and "identifiers".

Keywords are special words reserved by the language.
For example in python, some keywords would be: `def`, `if` and `while`.

Identifiers are typically names you assign to things you define yourself (e.g. variables, functions, classes).
For example in python: `MyClass`, `boban673`, `THE_MIGHTY_BOBAN`.

Keywords usually follow stricter rules than identifiers, for example they might be purely lowercase letters,
whereas identifiers usually can be uppercase, have digits and also underscores.

What constitutes a keyword or identifier depends on the settings in your current buffer.
You can view them by doing:

```vim
set iskeyword?
" For me, it prints
" iskeyword=@,48-57,_,192-255

set isident?
" For me, it prints
" isident=@,48-57,_,192-255
```

The format vim prints these results in is explained in detail in the appendix at the end of this kata.
A quick summary of the above is that it's a comma separated list of allowed character groups:

- `@` - ascii letters (English letters and some European letters like à)
- `48-57` - characters 48-57 (which corresponds to the letters 0-9)
- `_` - underscore
- `192-255` - characters 192-255 (which corresponds to some oddball extended ascii characters like ╬)

So `abc123_╬` would be considered a keyword and identifier under these settings.

In my case the settings are identical,
potentially because I'm in a markdown file which doesn't distinguish between keywords and identifiers.
The values were probably set by the markdown plugin I'm using.
If you ran this in a python or scala file you'd hopefully get something different.

# How does this relate to regexes?

Vim provides `\k` and `\i` for matching keywords and identifiers respectively.
Their behavior is defined by `iskeyword` and `isident` above.

Note as well there are `\K` and `\I` which are the same as their lowercase brethren, but they exclude digits.
This is handy for a lot of languages which don't allow digits as the first character for keywords or identifiers.
For example a keyword would be `\K\k*`.

If you're writing plugins, custom commands or defining syntax highlighting rules, then these atoms are very handy.

# Files

Vim also has a concept of filenames and the characters that are allowed in a path.

It's controlled by the `isfname` setting:

```vim
set isfname?

" On my system:
" isfname=@,48-57,/,.,-,_,+,,,#,$,%,~,=
```

There's corresponding regex atoms `\f` and `\F` like above.

## Using `isfname`

Vim uses `isfname` internally for file related operations.

For example when your cursor is on a file path, pressing `gf` causes vim to try and open that file in a buffer.

It will use `isfname` to determine how to walk left and right from your cursor to build out the text area
corresponding to the path.

# Exercises

See the [setup guide](advanced_regex_exercises_setup.md).

Further, to put us all on the same page, we'll explicitly set what characters constitute keywords, identifiers and paths:

```vim
" Set keywords to be just English alphabetical characters or underscore or asterix
set iskeyword=a-z,A-Z,_,*

" Set identifiers to be ascii letters (that permits German, Italian etc... letters), or digits or underscore
set isident=@,48-57,_

" Set filename characters to ascii letters, digits and a bunch of misc path characters
set isfname=@,48-57,/,.,-,_,+,,,#,$,%,~,=
```

Copying that out by hand will be a bit tedious. A shortcut would be to put your cursor on the the setting name above
(e.g. `iskeyword`) then do `:set <c-r><c-a><enter>`.
Remember `<c-r><c-a>` inserts the big word under the cursor.

Another way would be to just yank the line with `Y` then do `:<c-r>"<enter>`.
Remember `<c-r>[REGISTER]` copies text from the register.

Note in the above we included `*` as a keyword.
That's just to have a character that's a keyword but _isn't_ an identifier to make the examples more interesting.

## Exercise 1 - identifiers and keywords

Below we'll search for keywords and identifiers using patterns `\K\k*` and `\I\i*`.

As usual we'll limit the search to a block using `%V`.

- visually select the text between the `---` dividers then hit escape
- search `/\v%V\K\k*` (all the words from columns 1 and 3 should fully highlight)
- search `/\v%V\I\i*` (all the words from columns 1 and 3 should fully highlight)

```
Searches:
  \K\k*             (just keywords - English letters, underscore and asterix)
  \I\i*             (just identifiers - ascii letters, digits, underscore)

Match both         Just keywords             Just identifiers        Neither
------------------------------------------------------------------------------
boban                 *boban*                 böban                    =
boban_jones           joban*                  à                        ++
while                                                                  30
if                                                                     *30*
for
------------------------------------------------------------------------------
```

There were some partial highlights. For example the "b" and "ban" from "böban" would light up when searching keywords.
In the next exercise we'll make the words only light up if they're complete matches.

## Exercise 2 - no partial matches

As mentioned, we only want to see matches on entire words.

For example we don't want the "boban" inside "*boban*" to light up when searching for identifiers.

You might be thinking we could achieve this with word boundaries `<` and `>` but that won't work.
If you search `\v%V<\K\k*>` it will still match the "b" and "ban" from "böban" as 'ö' is causing a word boundary.
(This comes from the fact that it's not a keyword character under the current definition)

The boundaries we're after are whitespace boundaries. We only want a sequence of characters to light up if:

- to the left of the match is the start of the line or whitespace (positive lookbehind)
- the non-whitespace characters all match our condition (e.g. `\K\k*`)
- to the right of the match is the end of the line or whitespace (positive lookahead)

That means wrapping our core pattern with a lot of dressing:

```
Core pattern:  \K\k*                                                      \v - very magic
Wordy pattern: \v    %V    (^|\s)@<=    [CORE PATTERN]    ($|\s)@=        %V - restrict to visual
                                                                          (^|\s)@<= - positive lookbehind
                                                                          ($|\s)@=  - positive lookahead
```

We can use our programming chops from the previous kata to build this dressing around a core pattern.

We'll assume the core pattern in register `p`. Put your cursor on the nnoremap command below
and do: `Y:<c-r>"<enter>`

```vim
nnoremap ? :execute '/\v%V(^\|\s)@<=' . @p . '($\|\s)@='<enter>
```

Note that we had to escape the pipes above.
This is because they are separators in vimscript, analogous to semicolons in java or C.

Let's do some matching!

- highlight the text between the delims as usual
- do `:let @p = '\K\k*'<enter>`
- do `?` (note no partial highlights from column 3) 
- do `:let @p = '\I\i*'<enter>`
- do `?` (note no partial highlights from column 2) 

```
Match both         Just keywords             Just identifiers        Neither
------------------------------------------------------------------------------
boban                 *boban*                 böban                    =
boban_jones           joban*                  à                        ++
while                                                                  30
if                                                                     *30*
for
------------------------------------------------------------------------------
```

You could load other patterns into the `p` register and run other searches.

## Exercise 3 - filenames

For completeness we'll test out the `\f` and `\F` atoms for matching filename characters.

We'll use the same `?` trick from exercise 2, but load `\F\f*` into register `p`.

- highlight the block below as usual
- do `:let @p = '\F\f*'<enter>`
- do `?`
- do `:let @p = '\f+'<enter>` (allowing leading digits)
- do `?`

```
Is filename           Is not \F\f* filename
--------------------------------------------
~/home/boban          ~/^^&
/etc/ssh/ssh_config   0.md
foo
demo.sc
böban.text
--------------------------------------------
```

Note that it's usually fine for filenames to start with digits.
All the kata in this repo are proof of that!
So usually you'd just be searching for `\f+`.
We were just playing around with `\F` to check that it would make `0.md` as not being a file.

# Appendix - understanding the characters sets

When we do `:set iskeyword?` we get a comma separated list like:

```
@,48-57,_,192-255
```

The full explanation for this format is provided in `:help isfname` (scroll down a bit).

Scroll down to this part:

> The format of this option is a list of parts, separated with commas.
> Each part can be a single character number or a range.
> A range is two character numbers with '-' in between.
> A character number can be a decimal number between 0 and 255 or the ASCII character itself
> (does not work for digits).
> ...

The gist is that you can put in literal characters or their ascii code points as digits,
and ranges of those.

There's a few edge cases like "how do you specify comma in a comma separated list?"
The answer is to just put 3 commas in a row. The outer two are the delimiters.
We did this with `isfname`:

```
set isfname=@,48-57,/,.,-,_,+,,,#,$,%,~,=
"                            ^^^
```

# Conclusion

If you're defining syntax highlighting rules for kewords and identifiers then this section will be useful.

For general editing, you're probably not going to be searching for vague identifier characters when you
have a specific word you're after.
For example you'd search `for` not `\k{3}`.
