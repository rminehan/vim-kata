# Advanced regex 7 - multiline patterns

In today's kata we'll look at "multiline" patterns, that is how to find matches that cross line boundaries.

Along the way we'll introduce the buffer start and end anchors.

With that newfound knowledge we'll improve our whitespace cleaning script introduced in [kata 40](040_scripts.md).

# Can vim search across lines?

If we search the text below for the pattern `ab`, it won't find anything even though the first line
ends in 'a' and the second line starts with 'b'.

```
This line ends in a
b starts this line
```

The reason for it not matching is because there's a newline character after the 'a'. The string in really:

```
This line ends in a\nb starts this line
```

How about `a.b`? That doesn't work either.
You might remember mentions from previous kata about how `.` doesn't match the newline.

How about `a\sb`? (Recall `\s` means "whitespace") That _also_ doesn't work, even though the newline character
would usually be considered whitespace.

This can start to give you the false impression that the regex engine breaks our text up into lines,
and searches individual lines for matches.
If that was the case, then you wouldn't be able to have patterns match across multiple lines.

This is incorrect though, the regex engine itself isn't "line based",
instead a lot of the primitive atoms that we commonly use are biased to be "line based".
For example `.` and `\s`.

To convince yourself of this, search `a\nb` against the text above and you'll get a match.

# Newline variations

In vim, atoms like `.` and `\s` deliberately don't match on newline characters.
To understand why, have a read through [this argument](why_avoid_newline.md).

Usually this is the behavior that we want because we're in a text editor with a "line based" frame of mind.

But what do we do for those times when we actually _do_ want to cross line boundaries?

## Underscore based variations

Vim provides a newline matching variant for most atoms.

They're in the form `\_x` (x is just a placeholder here).

For example:

- `\_.` matches all characters (including newline)
- `\_s` matches all whitespace (including newline)

## Unexpected variants

There's even newline based variants that you wouldn't usually associate with whitespace.

For example:

- `\_d` - matches digits and newline
- `\_a` - matches letters and newline

Even `\S` (not whitespace) has a newline analog: `\_S` (not whitespace and newline)!

These variants would be useful for matching chunks that have been broken across lines.

For example `\_d+` would be useful for something like the below if it's supposed to be interpreted as one number.

```
0123
4567
890
```

Just remember that there's still newlines in your match, you won't be able to immediately parse it to an integer.

## The full list

To see the full list, run `:help /\_` and you'll get something like:

```
         /\_ E63 /\_i /\_I /\_k /\_K /\_f /\_F
         /\_p /\_P /\_s /\_S /\_d /\_D /\_x /\_X
         /\_o /\_O /\_w /\_W /\_h /\_H /\_a /\_A
         /\_l /\_L /\_u /\_U
\_x   Where "x" is any of the characters above: The character class with end-of-line added
```

## Always backslach

Even in very magic mode, you need the backslash, e.g. you can't do `_s` for "any whitespace or newline" - you need `\_s`

This applies to `\_.` too.

This rule makes sense as underscore is a common literal character.

# Beware the ravenous underscore dot

The atom `\_.` is a ravenous beast devouring all characters.

If you put a greedy quantifier like `+` or `*` on it, it becomes unstoppable and will consume your entire buffer
as it's no longer stopped by newlines.

Even `%` operators like `%V` won't stop it once it's started, as they only apply at the point they're scanned.

To make the point, set up a search with the block below and do `/\v%V\_.*` and you'll see it leaves the
search area and runs to the end of the buffer. This is because `%V` is only validating that a match _starts_ in the
highlighted area. See also the section on prison breaking in [this article](dangers_of_visual_mode.md).

Putting another `%V` at the end will "fix" this in the sense that only characters in the block will get highlighted.
However the regex engine still would have run to the end of the buffer and then backtracked character by character until
finally coming back into the search area. Depending on what you're doing this could be a big performance issue.

```
abc
def
ghi
```

# Don't forget newline itself

You can also just search explicitly for `\n` if you want to match the newline.

e.g. `/o\nb` will match below:

```
bo
ban
```

# Start and end of buffer

Recall from [kata 43](043_advanced_regex_1_restricting_search_space.md) we introduced `%` based atoms (like `%V`)
which were "context" or "buffer" oriented.

For example:

- `%V` means in the most recent visual selection
- `%45l` means on line 45
- `%20c` means in column 20

Today we're introducing more buffer related atoms:

- `%^` - start of buffer
- `%$` - end of buffer

These will be useful later for our whitespace cleaning.

# Exercises

See the [setup guide](advanced_regex_exercises_setup.md).

## Exercise 1

Join all the lines below such that trailing whitespace from the upper line and leading whitespace on the lower line
get combined and squashed into a single space.

We'll attack this with a `substitute`:

- put your cursor on the first line of text
- do `vip` then `jj`
- do `:s/\v\s*$\_s*%V/ /<enter>`

```

  Join me		
    with the  	
  the lines
below

 and this one


```

Some notes:

The `$` is necessary to make sure we're only finding matches that start at the end of lines.
Without it we'd be also finding the whitespace before "Join" and any multi-space blocks between words on the same line. 

Having the `$` in the pattern syntactically breaks the whitespace up into two groups - those before it and those after.

To make the regex easier to understand we've used regular `\s*` for the first group.
We could have actually used `\_s*` for that first group and it would work the same,
but the `$` would have a slightly different effect.
The `\_s*` could run over multiple newlines and the `$` only needs to match the last one.
This would cause some backtracking.

In contrast, the greedy `\s*` isn't inefficient here because it would get stopped at the end of the line
and won't overshoot.

The `\_s*` will match the newline and may run over multiple consecutive whitespace only lines:

```
  Join me\t\t\n    with the  \t\n  the lines\nbelow\n\n and this one\n```
                                                   ^       zero width match
                                                   -----   \_s* matches both newlines and the space before "and"
```

If we did want to use `\_s*` for both groups we still can't combine them syntactically because
unfortunately regex doesn't have an easy way to say:
"some whitespace characters that cross the newline boundary at some point".

The `$` is a zero width match _before_ the newline. That's why it's the second whitespace group which has the `\_s`.

Finally there are two small bugs in our pattern which are cancelling each other out.
See the detailed analysis in [this article](dangers_of_visual_mode.md) for more info. a small bug in our pattern 

## Exercise 2 - prison break

In exercise 1, the pattern we used had a `%V` at the end.

This was to stop a kind of prison break by the regex engine.
It's easiest to understand by running the exercise again without it.

Note though it will mess up the backticks in the block and probably your formatting
so you'll want to undo after the substitute.

- put your cursor somewhere in the first paragraph
- do `Vipjj`
- do `:s/\v\s*$\_s*/ /<enter>`
- do `u` to restore the block

```

  Join me		
    with the  	
  the lines
below

 and this one




```

What you would have ended up with would be:

```

  Join me with the the lines below and this one [BACKTICK][BACKTICK][BACKTICK]
```

Notice how it collapsed newline characters that weren't in the original visual selection.

This is because the newline after "one" was matched _within_ the visual selection and that
began a match which then broke out of the original selection.

It ran all the way to the backticks replacing all the newlines from after "one" with a space.
Hence the backticks end up on the same line as the other text.

The point of this exercise is to make you realize that `\_*` style atoms can cause prison breaks.
Prior to this it's never been an issue you've had to deal with because newlines act like a fence
which all the standards atoms don't match.

More info on prison breaks [here](dangers_of_visual_mode.md).

## Exercise 3 - naughty n

`n` (for next match) doesn't quite do the right thing with multiline matches and it can be confusing.

We'll write a pattern which will create a single match in the text below, but we'll see `n` will act like
there's 3 matches.

- prepare the search area as usual
- do `/\v%V\_w+<enter>` (`\_w` means wordy characters and newline)
- hit `n` a few times - if your vim is like mine it will cycle between the starts of each line of a's

```
aaa
aaa
aaa
```

So how many matches are there?

There's 1 big match. `\_w` will match from the first 'a', through all the a's and newlines and stop at the tildas.

To see this let's ask `substitute`:

- visually select the area above
- do `:s/<c-r>//x\r/<enter>`

Remember `<c-r>` followed by a register inserts from that register.
In our case the `/` register is our last search.
So this was just a show-off way to put the last search into our substitute.

We're replacing all matches of our pattern with `r<newline>`.
The newline is to stop the tildas riding up on to the previous line as the match includes all the newlines in the block.

You'll see just one `x` appear meaning that `substitute` found just one match.

So what's going on? I think `n` just searches each line individually for matches in isolation.
To prove that:

- prepare the block below
- do `/\v%Va+\nb+`
- hit `n`

```
aaa
bbb
```

Note that `gn` introduced in [kata 38](038_search_text_object.md) seems to work properly.

Note that this might just be a bug in neovim that gets fixed at a later point.
My version is 0.4.4.

## Exercise 4 - better whitespace cleaning

In [kata 40](040_scripts.md) exercise 2 we introduced a simple vimscript to clean up whitespace in the current buffer:

```vim
" Remove trailing whitespace
" After this we can assume any blank lines are empty
% substitute /\v\s+$//

" Replace all consecutive newlines with two newlines
" This will compress multiple blank lines together
" if they're in the middle of the buffer, but at the
" start and end it may still leave 2 blank lines
% substitute /\v\n\n+/\r\r/

" Remove the first line if it's blank (twice in case of 2 blank lines)
1 global /^$/delete
1 global /^$/delete

" Remove the last line if it's empty (twice in case of 2 blank lines)
$ global /^$/delete
$ global /^$/delete
```

The section finishes with a mysterious reference to more advanced regex:

> There are more clever ways to remove the leading and trailing blank lines that involve using multiline regexes.
> Being so tricky they're beyond the scope of this kata but we might revisit this in a later kata on complex regex.

Well as Moss says, strap seatbelts to your ears because they're about to be taken for the ride of their lives.

This little script does the trick:

```vim
" Remove all trailing whitespace going as far as possible.
" When a non-whitespace is hit, backtrack to the end of the last line
" to prevent removing whitespace at the start of a line.
% substitute /\_s*$//

" There is still potentially a blank first line.
1 substitute /\n//g
```

The first command does most of the work.
It's similar to the one from previous exercises but without the `%V` stuff as it doesn't make sense for whole buffers.

The first command always leaves a newline behind after non-blank lines.
This prevents joining non-blank lines.
The `$` always matches _before_ newlines, so it acts like a guard here making sure that `\_s*` always has
to backtrack and stop before the last newline.

Note the `\_s*` might greedily consume multiple blank lines.
The newline that the `$` protects will be the newline before the next non-blank line:

```
Protect


the newline!
```

In 1D:

```
Protect\n\n\nthe newline!
       -->-->              \_s*
           <-              backtracks until $ is satisfied
           $               protects the last newline from consumption
       ----                final match - two newlines
```

If you have a blank first line(s), the first command will still protect a newline from that batch.
For example suppose the first 3 lines were blank:

```
\n\n\n# Title
-->-->
    <-
    $
```

The first two newlines will get deleted leaving:

```
\n# Title
```

So the second substitution just looks at line 1 and deletes a newline if it's there.

- move the vimscript above into a file `whitespace_clean.vim` and save it (you can quit).
- do `:new<enter>`
- fill it with random text with whitespaces
- do `:source whitespace_clean.vim<enter>` to test out the script
- do `:q!<enter>` to kill the split

## Exercise 5 (optional)

Suppose we wanted to only remove blank lines at the end of a file.

This is a good chance to use `%$`.

- do `:new<enter>`
- fill it with random text and some trailing blank lines
- do `:%s/\v\_s*%$/g<enter>`
- do `:q!<enter>` when you're done

The search is similar to the previous exercise except it uses `%$` instead of `$`.

Notice how `%$` still protects a newline. This is good for making sure files have a newline at the end.

# Conclusion

Multiline regexes have their uses (mainly in scripts) but because they're lesser known there are some gotchas with them.

For example beware of consuming your entire buffer using `\_.` with a greedy quantifier.

There are also some subtle bugs related to visual mode (see [related article](dangers_of_visual_mode.md)).

They can also be confusing because we're used to line based thinking,
so if there's a simple line based way to do something,
consider using that approach instead if you're planning on sharing it around,
or even just for your future self reading your script later on.
