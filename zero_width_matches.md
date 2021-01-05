# Zero Width Matches

This mini blog is an aside from [kata 48](048_advanced_regex_6_ranges.md) on advanced regex ranges.

During some exercises, the pattern `\d{}` was being searched in very magic mode on text areas like:

```
012     123456      0

00      12345678   0x23456

9987    123456789
```

(Recall that `\d{}` is identical to `\d*`, "zero or more digits")

If you run the search you'll see the whole block light up and it might have made it look like every line was a
complete match.

That doesn't make sense though as the pattern is about digits.

Rerun the a similar search using a pattern like `x{}` (zero or more x's) and you'll see the same thing.

So it's not really about digits - it's about the "zero or more" quantifier and how vim displays matches.

# What's actually happening?

If you lock in the `x{}` search and hit `n` you'll notice that there's actually a match at every location.

Even though there are no 'x' characters in the block, the regex engine still finds zero x's at every position
which is consistent with "zero or more".

Each match is "zero width" though - it doesn't actually consume the character it's on.

So there is a match _before_ every character or zero width.

Vim's way of highlighting zero width matches is to highlight the character following the match.
So it looks identical to if that character had been _consumed_ by a match.

Further vim doesn't display the boundaries between adjacent matches.
So at first glance it looks like every line is a single match.

# Replacing zero width matches

Visually select the block below and do a substitute to replace every `x{}` with 'Z'
by running `:s/\vx{}/Z/g<enter>`

```
0123
012
01
0
```

You should get:

```
Z0Z1Z2Z3
Z0Z1Z2
Z0Z1
Z0
```

Notice how the original characters weren't replaced.

Instead a zero width area before each character (including the spaces) was replaced with 'Z'.

The effect is that a 'Z' was _inserted_ before each character.

If there had been actual 'x' characters in the text we'd get regular replacement for those.

# Line anchors

Another simple example of zero width matches are anchors.

Visually select the below and do `:s/^/START/<enter>`:

```
Line 0
Line 1
Line 2
```

It "replaces" the zero width line start anchor (`^`) with "START".
The 'L' starting each line is not affected.
So this is really the same as just inserting "START" at the start of each line.

Some other examples of zero width anchors are:

- end of line `$`
- the cursor `%#`
- start of file `%^`
- end of file `%$`

# Original example

The original example was running `\d{}` against:

```
012     123456      0

00      12345678   0x23456

9987    123456789
```

It's more complex because there's a mix of zero width and non-zero width matches.

- each block of digits is a non-zero width match
- every non-digit character has a zero width match before it

To understand it better in the context of a replace, we'll use a smaller example.

Visually select this block and do `:s/\v\d{}/Z/g<enter>`:

```
00 11 22
00 11
00
```

You should get:

```
Z Z Z
Z Z
Z
```

This might not be what you expected. For example on line 1, you might have expected:

- "00" is a match and should become 'Z'
- the first space has a zero width match so should have a 'Z' inserted before it
- "11" is a match and should become 'Z'
- the second space has a zero width match so should have a 'Z' inserted before it
- "22" is a match and should become 'Z'

So shouldn't line 1 become: "ZZ ZZ Z"?

It seems here that the zero width matches for the spaces got combined with the non-zero width matches behind them.

Hence we just end up with 3 replacements for "00", "11" and "22".

The block below has _2_ spaces between each chunk of digits.
Visually select this block and run the same substitution:

```
00  11  22
00  11
00
```

you should get:

```
Z Z Z Z Z
Z Z Z
Z
```

Now line 1 has 5 Z's. Our hypothesis is:

- each block of 2 spaces produced two zero width matches
- the zero width match for the first space got combined with the non-zero width match from behind it.
- the zero width match for the second space wasn't combined
- hence a 'Z' got inserted before the second space, but not the first space

An alternative hypothesis is that zero width matches get combined with non-zero width matches _after_ them.
That would mean the extra 'Z' is coming from the zero width match before the _first_ space.
But then wouldn't we get "ZZ  ZZ  Z" for the first line?

This is a good chance to break out some ampersand regex fu to really prove this.
TLDR - our first hypothesis is right. So you can skip the next section if you want ot see `&` in action.

## Proving our hypothesis

### Methodology

We know that the extra 'Z's we're seeing are coming from zero width matches either from the first space or second space.

Hence if we disable the zero width match before the second space, and still see an extra 'Z', we know that
'Z' must have come from the first space.

We can do this by adding an extra condition to only match space characters when they have a space _after_ them.
The first space will satisfy this and the second won't.

Later when we flip it around, we'll only match spaces that have a digit after them.
This first space won't satisfy this, but the second will.

So our new match criteria will be:

- 1 or more digits (to create non-zero width blocks)  OR
- we're on a space with another space after AND 0 or more digits AND w

We substitute this for 'Z' and see what happens.

### Translating to regex

The top level "OR" logic is represented by `(....|....)` and the "AND" logic for the second point is represented by `&`.

The core of the pattern is:

```
(   \d{1,}   |   (  )   &   \d{}   )

<----------- OR ------------------->
   1 or more     <---- AND ---->
    digits        2        0 or more
               spaces       digits
```

You might want to review [kata 47](047_advanced_regex_5_ampersand.md) which introduced `&`. The crucial points here are:

- all `&`'d branches must hold starting from the same position (and may consume different numbers of characters)
- it's the last branch in the chain of `&`s which represents the match that the engine continues with
- hence early branches just act as extra conditions that must hold at that position

In our case we want a zero width match produced for the `&` side, so that we can see if it combines with non-zero
width matches before it. Hence we put the `\d{}` on the right of the ampersand.

This means the `(  )` branch is just a check: "Are there two spaces starting at this location?"

## Testing out the first space

Visually select this and do `:s/\v(\d{1,}|(  )&\d{})/Z/g<enter>`

```
00  11  22
00  11
00
```

You should get:

```
Z  Z  Z
Z  Z
Z
```

There were only 3 matches on the first line that got substituted.
We _didn't_ get the extra Z's appearing from zero width matches.

Our conclusion is that zero width matches from the first space are being combined with the non-zero width
matches preceeding them _before_ the substitute happens. 

## Confirming second space produces the extra Z's

To confirm our understanding, we'll do the same thing but for the second space.
Those are the spaces with a digit directly after them.

```
(   \d{1,}   |   ( \d)  &   \d{}   )
             OR
   1 or more           AND
    digits       space      0 or more
               followed     digits
               by digit
```

Visually select this and do `:s/\v(\d{1,}|( \d)&\d{})/Z/g<enter>`

```
00  11  22
00  11
00
```

you should get:

```
Z Z Z Z Z
Z Z Z
Z
```

_Now_ we're getting extra spaces as expected.

They are coming from the zero width matches associated with the _second_ spaces and _aren't_ being combined
with non-zero width blocks following them.

### Conclusion

So far it looks like during a `substitute`, zero width matches get combined with non-zero width matches that consume
characters right up to where the zero width match starts.

The effect is that you get less replacements.

Without finding any official docs though, we don't really know what's going on - it could be something more complex.

# Summary

Vim has many zero width match concepts.

For example `^`, `$` and quantifiers which allow zero matches, e.g.

- `x{,5}` (0 to 5 x's)
- `y{}`   (0 or more y's)
- `z{-}`  (0 or more z's - lazy)

When we search with quantifiers that allows 0 matches, characters in the search area that don't match
will still get highlighted because there's a zero width match before them (and that's how vim shows that).
This can create a lot of highlight "noise".

Likewise if you search for `/^` you'll notice the first character of each line getting highlighted,
even though the character itself isn't really being matched, it's the zero width match _before_ it.

Visually vim uses the same highlighting for consumed characters and characters with a zero width match before them.
So for a search like `\d{}`, everything will be highlighted and you can't clearly see the match boundaries
between number blocks and non-number blocks.
To see the individual matches, lock in the search and use `n` to jump between the results.

When _substituting_, zero width matches still get replaced which has the effect of the replacement
text being "inserted" as no characters were replaced.

However zero width matches seem to get combined with non-zero width matches that end just before them.

This is a case where running `substitute` will behave a little differently to manually applying a change
at each match using `n` and `.`.
