# Advanced Regex 1 - restricting search space

The next block of kata are on advanced regex patterns in vim.

Initially we'll just focus on general matching in a read-only way,
then look at specific replacing tricks.

Today we're looking at advanced tricks to restrict the search area for patterns.

# Motivation/Methodology

In kata's 11-13 we looked at buffer search which introduced vim's regex engine.

Most of the time fzf and basic buffer search will let you quickly find what you want without
needing to learn vim's more advanced regex features.

Nonetheless they can still be useful from time to time, particularly in the context of scripting,
both for writing your own scripts or trying to understand cryptic regexes from stackoverflow or other scripts.

Because you don't use advanced regex much, it's not realistic that you'll memorize them long term.
So the aim for this block of kata is just to make you aware of the existence of these tricks,
so you can find them when you need them.

And because it's used less, I've deliberately put it after fzf and other more practical search mechanisms.

# Regex flavors

There are many flavors of regex (e.g. perl, vim, python, java).

Syntactically, for common regex operations they'll tend to behave the same, but diverge on more obscure operations like:

- handling the newline character
- look ahead/behind

The engines themselves will also have differences like performance and implementation details like how to handle backtracking.
Vim itself actually has two regex engines and chooses which one to use based on the pattern.

Today and next time we're looking specifically at vim's regex so a lot of the syntax won't port to other flavors.
Also some of these features only make sense in the context of editing inside a buffer where you have editor concepts
like a cursor, marks, line and column numbers.

# Resources

I've put together an [advanced regex cheat sheet](advanced_vim_regex_cheat_sheet.md) as a reference guide.
It's designed for someone who already knows "day to day" regex (e.g. `.`, `*`, `+`, `[...]`)
and to reduce noise it just focuses on less common features specific to vim.

Vim also has a very comprehensive help page for its regex which lists all the nifty features.
You can view it in a browser [here](http://vimdoc.sourceforge.net/htmldoc/pattern.html) or read it inside vim with `:help pattern`.
It's not _too_ big and worth having a quick browse through.

# Exercises

For each of these advanced regex kata, there's some standard editor setup and notes.

Because there's so many kata that use it,
I moved it to [its own file](advanced_regex_exercises_setup.md) for quick reference.

So have a look!

## Exercise 1 - limiting the search area

Vim has some neat tricks for limiting the search area to specific portions of the buffer.

Line number based ones: (`:help /\%l`) (that's the letter 'l' ("el") not the digit '1')

- `%30l` - only line 30
- `%<30l` - before line 30
- `%>30l` - after line 30

Column based ones: (`:help /\%c`)

- `%20c` - only column 20
- `%<20c` - before column 20
- `%>20c` - after column 20

There is also `%V` (`:help /\%V`) which limits the match to the most recent visual selection
(probably using marks `'<` and `'>` under the hood).
Note the `%` - don't confuse this with `\V` for very-no-magic.

Overall think of `%` as meaning "context", as the operators that follow it tend to be related
to the buffer being searched (e.g. line numbers, columns, last visual selection, cursor position).

The block below has column markers for digits and tens (note that columns start at 1 in vim).

For this exercise, we'll find all the "bobans" contained entirely in columns 20-29.

- visually select the area between the dividing lines `----` then press escape
- put your cursor on the "boban" in column 25 (first row) (he's the only lowercase one)
- do `/\vboban` (all boban's in view should highlight due to case insensitivity)
- add `%V`
    - what is _supposed_ to happen is that all boban's in the block highlight
    - but depending on your vim, only the lowercase one may highlight
    - potentially the capital "V" is causing `smartcase` to disable case insensitivity (even though it's not a literal)
- do `\c` to explicitly turn on case-insensitivity (now all the boban's should match)
- add `%<31c` (now only the boban's that _end_ before column 31 should highlight)
- move your cursor back to just after the `\v` and add `%>19c`
    - now only boban's that start after column 19 should highlight

```
123456789012345678901234567890123456789
0        10        20        30        40
----------------------------------------------------
Boban    |         |    boban|         |
         |Boban    Boban     |         |
         |     Boban     Boban     Boban
         |         |  BOBAN  Boban     | BoBaN
         |        Boban  Boban         |
----------------------------------------------------
```

The final pattern is `\v%>19cboban%V\c%<31c`.
It's a beast but hopefully by building it interactively it's not too intimidating.

If all went well, it should highlight 3 matches starting in columns 25 ("boban"), 20 ("Boban") and 22 ("BOBAN").

Note the column conditions apply to the position the regex is up to.
So the `%>19c` is taking effect _before_ matching on the 'b/B' for each boban.
The `%<31c` is being tested after the engine has processed the full boban.

One thing that might confuse you is why we did `%<31c` and not `%<30c`.
Try it again with 30 and you'll see it won't pick up the "boban" starting column 25.
This is because when the regex engine has finished processing "boban" it's now on the character _after_ the 'n' (the pipe).
So it's already on column 30 when it processes the column condition.

Note that `%V` is acting like a zero-width anchor at a specific position (similar to the column conditions above).
It's not a "flag" like `\c` that applies to the whole regex.
In our pattern it's technically only asserting that the position after each 'n/N' for each boban is inside the visual selection.

In our case, because we used linewise visual selection and we're not doing multi-line regexes,
having one character in the visual area implies all characters are so we can be pretty loose about where it goes in our pattern.
In more complex situations though it's better form to put it at the start and end of your pattern if you want to make
sure the entire match is contained in the visual selection.

Note again the use of very magic mode `\v` here saved us a fair bit of backslashing on all those `%` signs.
But the docs and most examples will be for magic mode where escaping is necessary.

## Exercise 2 - line based filters

For this exercise we'll repeat the above example using line numbers instead of `%V`.

We'll use a pattern like `\v%>111l%<117l%>19cboban%<31c`:

```
\v%>175l%<181l%>19cboban%<31c       \v     - very magic           %>19c - after column 19
--      ------     -----            %>175l - after line 111       boban - literal "boban"
  ------      -----     -----       %<181l - before line 117      %<31c - before column 31
```

- determine the first line number and last line numbers for the two `---` delim lines below
- put your cursor on "boban" again
- `/\v` to start a very magic search
- add `%>` then put in the line number of the first `---` delim, then the letter `l`
- add `%<` then put in the line number of the second `---` delim, then the letter `l`
- add `%>19c` as in exercise 1
- add `boban`
- add `%<31c`

```
123456789012345678901234567890123456789
0        10        20        30        40
---------------------------------------------------- (get this line's number)
Boban    |         |    boban|         |
         |Boban    Boban     |         |
         |     Boban     Boban     Boban
         |         |  BOBAN  Boban     | BoBaN
         |        Boban  Boban         |
---------------------------------------------------- (get this line's number)
```

### Exercise 3 - marks

We'll repeat the exercise but use marks to mark the line numbers to search between. `:help /\%'m`

For example "after mark `a`" is:

- `\%>'a` in magic mode
- `%>\'a` in very magic mode (unfortunately we have to escape the `'`)

We'll put mark `a` on the first delim line and `b` on the second delim line.
Then the pattern will look like:

```
\v%>\'a%<\'b%>19cboban%<31c       \v    - very magic            %>19c - after column 19
--     -----     -----            %>\'a - after mark a's line   boban - literal "boban"
  -----     -----     -----       %<\'b - before mark b's line  %<31c - before column 31
```

- put your cursor on the first delim line and do `ma`
- put your cursor on the second delim line and do `mb`
- put your cursor on "boban"
- do `/\v` to start a very magic search
- add `%>\'a` to signal "after mark `a`" - note this will include some of the tail of the first delim line
- add `%<\'b` to signal "before mark `b`" - like above this will include some of the head of the second delim line
- add `%>19c`
- add `boban`
- add `%<31c`

```
123456789012345678901234567890123456789
0        10        20        30        40
---------------------------------------------------- (put cursor on this line and do `ma`)
Boban    |         |    boban|         |
         |Boban    Boban     |         |
         |     Boban     Boban     Boban
         |         |  BOBAN  Boban     | BoBaN
         |        Boban  Boban         |
---------------------------------------------------- (put cursor on this line and do `mb`)
```

The final pattern should be `\v%>\'a%<\'b%>19cboban%<31c`

## Exercise 4 - using built in marks

In the previous exercise we used marks `a` and `b` to specify a range.

Remember from [kata 28](028_built_in_marks.md) that visually selecting text automatically sets the `<` and `>` marks.

We'll combine this with exercise 3. The pattern will look basically the same:

```
\v   %>\'a   %<\'b   %>19c   boban   %<31c   Exercise 3
\v   %>\'\<  %<\'\>  %>19c   boban   %<31c   Exercise 4 - very magic mode
     \%>'<   \%<\'>  \%>19c  boban   \%<31c  Exercise 4 - magic mode
```

Again sadly we have to escape the carets in very magic mode otherwise they're understood as word boundaries.

For this one, we'll use regular magic mode (the default) just to mix things up and make the use of marks a bit easier
to understand. It does mean we have to escape the `%`'s.

- visually select the block below then hit escape
- put your cursor on "boban"
- start a regular search with `/`
- add `\%>'<` to signal "after mark `<`"
- add `\%<'>` to signal "before mark `>`"
- add `\%>19c`
- add `boban`
- add `\%<31c`

```
123456789012345678901234567890123456789
0        10        20        30        40
---------------------------------------------------- (put cursor on this line and do `ma`)
Boban    |         |    boban|         |
         |Boban    Boban     |         |
         |     Boban     Boban     Boban
         |         |  BOBAN  Boban     | BoBaN
Boban    |        Boban  Boban         |
---------------------------------------------------- (put cursor on this line and do `mb`)
```

The final pattern should be `\%>'<\%<'>\%>19cboban\%<31c`

This isn't quite the same as `%V` from exercises 1 and 2.
Do `/\%>'<` again and notice how it excludes the `B` of the top left "Boban".
This is because `'<` represents the first character of the first line and we're searching _after_ that.

Compare these searches:

```
/^\%>'<\%<'>Boban        (1 match)
/^\%VBoban               (2 matches)
```

Likewise there could be some edge cases at the bottom right of the visual selection.

# Conclusion

Advanced regex is powerful but usually not used enough to lock it into your brain.

Hopefully today has made you aware that you can limit the area for matches even if you can't remember the exact syntax.

From there you can look at the cheat sheet or use vim's built in `:help pattern` to get you what you need.

This is just the beginning of our deep dive into advanced regex.
Going forward we'll particularly lean on `%V` from today to limit search patterns in later exercises to just
the text block related to the exercise.
