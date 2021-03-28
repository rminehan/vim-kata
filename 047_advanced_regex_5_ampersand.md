# Advanced regex 5 - ampersand

Ampersand provides a way for you to specify multiple branches in a regex pattern.

This can be useful for modelling complex non-linear logic in a pattern.

There's not that many use cases for ampersand that can't be solved with look arounds.
But one case where it's useful is for logic related to the cursor position.

# Ampersand

Ampersand represents the logical "and"ing of two patterns in that they both must hold at the same starting point.

In very magic mode, it's `&` and in magic mode downwards it's `\&`.

## Example

Highlight the block below as usual, then do `/\v%V(Boban)&(\D{3})`.

(here `\D` means "not digit", so `\D{3}` means "3 not digits").

```
Boban-Jones
```

You should see it match "Bob".

Now do `/\v%V(Boban)&(Jones)`

It shouldn't match anything.

In the first example, the engine found that both the patterns `Boban` and `\D{3}` are satisfied starting at the 'B'.

In the second example, there's no location in the string where both `Boban` and `Jones` match.

## More complex example

You can and together multiple branches.

Highlight the block below then do `\v%V(Boban)&(\a{4})&([^a]{3})`

```
Boban
```

This is 3 patterns:

- "Boban"
- 4 alphabetical letters
- 3 characters that aren't 'a'

It should match "Bob".

Notice that the 3 patterns all match at the same starting point ('B') but they cover different ranges of characters,
and the last one seemingly contradicts the other 2.

All that matters is that each individual pattern can be matched in isolation from the same starting point.

## What gets matched then?

If you have mutiple patterns all running from the same point, and each potentially covers different parts of the text,
then what is the final block of text considered "matched".

For example if this pattern were being used in a find-replace, which of the branches actually gets replaced?

The answer is the last one.

That's why "Bob" matched in the previous example - it was the 3 characters that aren't 'a'.

What this means is that the patterns before the last pattern are really just there to act as filters or checks. 
They are playing a similar role to positive lookaheads in that they're not matched, but they can assert some condition.

# Cursor position

In the first advanced regex kata we introduced several `%` based operators for restricting matches to certain
areas of the buffer, e.g. `%V`, `%>20l`, `%<31c`.

Today we're introducing `%#` which is a zero width atom representing the current cursor. 

Put your cursor at the start of one of the "Boban"s below and do `/\v%#Boban`

```
Boban
Boban
Boban
```

It should only match the "Boban" your cursor is on.

If you move your cursor to the start of another "Boban" and hit `n` that should show that the new "Boban" is now the match.

If you move your cursor off the start of the word and hit `n` you should see it telling you there's no matches.

# Exercises

See the [setup guide](advanced_regex_exercises_setup.md).

## Exercise 1

In the warmup above, we showed how to match "Boban"s if the cursor (`%#`) was at the start.

What about matching a Boban that has the cursor _anywhere_ in the "Boban".
We know that all the characters in "Boban" are letters - `\a`. `:help /\a`

We can achieve this with an "and":

- branch 1: match any number of `\a`, then the cursor
- branch 2: match any old "Boban" (with word boundaries)

Branch 2 limits us to just "Boban"s.
Branch 1 limits us to any position where there's only letters between that position and the cursor.

"And"ing those two together should _only_ match a Boban with the cursor somewhere on it (or just after it).

Note we've put "Boban" on the second branch because we want the final result of the match to be "Boban",
rather than the characters before the cursor.

- highlight the block below as usual
- put your cursor anywhere in the block below
- do `/\v%V` to start a very magic search in the block
- add `()&()` (we'll fill in the gaps - full rationale below)
- in the second bracket group, put `<Boban>`
- in the first bracket group, put `\a*%#`
- move your cursor to different places and trigger the search with `n`
    - (which ironically will move your cursor to the start of a line)

```
Boban  
Boban  
Boban  
```

After each line there's 2 whitespace characters. Also test them out with `n`.

You _should_ find that the whitespace just after "Boban" still matches.
This is because there's any number of alphabetical characters _before_ it.

The second whitespace though _shouldn't_ generate matches.

If you don't like this behavior, add `\a` to the back of the cursor condition.
This makes sure there's at least one alphabetical character on or after the cursor.

The final patterns would be:

```
# Match if the cursor is on "Boban" or just after
\v%V(\a*%#)&(<Boban>)

# Match if the cursor is on "Boban"
\v%V(\a*%#\a)&(<Boban>)
```

The rationale for typing out `()&()` is a couple of things:

- it makes the big picture a bit clearer for people following along - it shows the main structure is an `&`
- it means the the pattern is parseable more often whilst you build it

To expand on the second point, if you start typing `(\a*.....` because the bracket isn't closed,
vim will deem the pattern invalid and you'll get no feedback about what's going on until the bracket finally gets closed.
You'll have less intuition for what effect each key press is having on the pattern.

## Exercise 2

We want to find all the lines below that have both "Boban" and "Enxhell" in them, but we don't care about the order
those two terms appear in the line.

For the lines that have both, we're wanting to match the "Enxhell" text.

```
Boban and Enxhell
Enxhell and Boban
Boban and Boban - double trouble
Enxhell and Enxhell - the bots have multiplied!
Lonely Boban
Lonely Enxhell
```

An initial approach could be: `\v%V(Boban|Enxhell).*(Boban|Enxhell)` which is:

- "Boban" or "Enxhell"
- any old text
- "Boban" or "Enxhell"

But this will pick up lines like "Boban and Boban - double trouble".

This basically fixes it: `\v%V(Boban.*Enxhell|Enxhell.*Boban)`.
The downside here is the duplication in the pattern,
which is not a huge deal, and we can even build the pattern using vimscript in a DRY way.

We also need to make some changes to make the "Enxhell" part the only text matched.
One way would be to mark all the other parts as look arounds:

```
\v%V((Boban.*)@<=(Enxhell)|(Enxhell)(.*Boban)@=)
```

Now it's a bit more unwieldy.

Another approach would be to use ampersand which we'll do for this exercise:

- highlight the text area below as usual
- do `/\v%V` to start a very magic search on that text area
- add `()&()` (same rationale as in exercise 1)
- go back to the first bracket group
- fill it with `.*Boban`
- go to the second bracket group
- fill it with `.*Enxhell`
- (at this point we've got the right lines, but the overall match needs to be reduced to just "Enxhell")
- put `\zs` before "Enxhell"
- for good measure put `\ze` after "Enxhell"

```
Boban and Enxhell
Enxhell and Boban
Boban and Boban - double trouble
Enxhell and Enxhell - the bots have multiplied!
Lonely Boban
Lonely Enxhell
```

The final pattern should be:

```
\v%V(.*Boban)&(.*\zsEnxhell\ze)

Broken down:

\v  %V  (.*Boban)  &  (.*\zsEnxhell\ze)
                       .*  \zs  Enxhell  \ze


\v - very magic                       .*Boban - any characters
                                                then "Boban"
%V - inside visual
                                      .*Enxhell - any characters
(...) & (...) - both must match                   then Enxhell
                at the same position
                                      \zs...\ze   - zoom anchors
                                                    whatever's between
                                                    them is the final match
```

So what happened here?

Ignoring the `\v%V`, the pattern is an `&` of two branches.

The engine will try to find a position where both branches match.

The first branch is `.*Boban` and will match from the start of the line on any line that contains "Boban".

The second branch is basically the same: `.*Enxhell` (if we just ignore the zoom anchors for now).
It will match from the start of the line on any line that contains "Enxhell".

So the `&` will see that both branches match from the start of the line.

The problem we have now is reducing the match to just the "Enxhell" text (and not everything up to it too).
Could we use a positive lookbehind to scrub out the `.*` before "Enxhell"?
Try it and you'll see that it now only matches "Enxhell and Boban".

To understand why this is, remember that the regex engine is trying to find a position where _both_
`.*Boban` and `.*Enxhell` match.

The pattern `(.*)@<=Enxhell` will start matching _at the 'E'_ of "Enxhell" but not from the start of the line:

```
Boban and Enxhell
          ^ (.*)@<=Enxhell matches here  :)
          ^ .*Boban doesn't match here   :(
          & fails at this position

^ (.*)<=Enxhell doesn't match here :(
^ .*Boban matches here             :)
& fails at this position
```

The result is that in "Boban and Enxhell", there's no position where both branches work.

In "Enxhell and Boban" they do though because "Boban" is after "Enxhell":

```
Enxhell and Boban
^ (.*)<=Enxhell matches here :)
^ .*Boban matches here       :)
& succeeds at this position
```

So this is one of those cases where a look around doesn't work for us because of the ampersand.
The crux of the issue is that we actually need the `.*` before Enxhell to be "consumed" by the engine so that both
branches "start" at the same location (the start of the line in this case).

Thankfully for a case like this, vim has handy "zoom" anchors `\zs` (zoom start) and `\ze` (zoom end).
These are quick and dirty ways to set what text we want to match.
See also `:help /\zs` and `:help /\ze`.

In our case we only needed to set `\zs` because everything after that which was matched is just "Enxhell".
We added `\ze` just for completeness.

Summarizing this epic exercise, here are two approaches:

```
# Regular search
\v%V((Boban.*)@<=(Enxhell)|(Enxhell)(.*Boban)@=)

# Ampersand based approach
\v%V(.*Boban)&(.*\zsEnxhell\ze)
```

The second has less nesting, no repetition and no look arounds which makes it easier to get the gist of it.
It will probably take some pondering though for the average reader to understand why it works
as it requires understanding the subtleties of the `&` operator.

Other factors to consider would be relative performance if it was being used at scale.

# Conclusion

`&` is an advanced regex feature which allows more sophisticated forms of conditional logic.

It's not well known, so if you use it people might have trouble understanding your patterns.

However there might be cases where not using it requires building something quite ugly and equally buggy
and hard to understand.
