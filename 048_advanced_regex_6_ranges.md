# Advanced regex 6 - ranges

Today we'll look at regex ranges, which also introduces the concepts of lazy quantifiers.

A regex range in vim is a quantifier where you specify a range of numbers. You use them to express concepts like:

- 3-5 digits             (left and right bounded)
- 4 or more letters      (left bounded only)
- at most 6 digits       (right bounded only)

See `:help /\{`.

Each of these has a greedy/eager and lazy mode.

This is covered more comprehensively in the appendix.

# Exercises

See the [setup guide](advanced_regex_exercises_setup.md).

## Exercise 1

Find all the groups of 2+ digits preceeding a "Boban". Match just the digits.

- highlight the area as usual
- do `/\v%V` to start a very magic search on the search area
- add `\d{}` (this will seemingly highlight everything - see the [zero width](zero_width_matches.md) article)
- inside the braces put `2` (this should narrow the matches to just those with exactly 2 digits) 
- add a `,` after the `2` (this should widen it to those with 2+ digits
- after the braces, add `Boban` (this should reduce it to just digits preceeding "Boban"s)
- put `(...)` brackets around `Boban` then add `@=` to make it a positive lookahead

```
Boban        34Boba

46Boban      9Boban9

Boban01      456Boban
```

The final pattern should be:

```
\v%V\d{2,}(Boban)@=               \d{2,}     two or more digits
                                  (Boban)@=  positive look ahead of Boban
\v  %V  \d{2,}  (Boban)@=
```

## Exercise 2

Find all the groups of 3-6 digits, matching as many as possible.

- highlight the area as usual
- do `/\v%V` as usual
- add `\d{}` like before
- inside the braces add `3` - now it will match groups of 3
    - you'll get multiple matches in some words below
- add `,` - now it will match groups of 3+, so at most one match per word
- add `6` - now it will match groups of 3-6 greedily
    - again some will have multiple matches in the group

```
012     123456      0

00      12345678   0x23456

9987    123456789
```

The final pattern should be `\v%V\d{3,6}`

Some words have multiple matches back-to-back which can be hard to see depending on how vim highlights matches.
For example "123456789" will have "123456" and "789".
Lock in the pattern and use `n` to jump around the matches to make it clearer.

Notice that the "12345678" only got one match - "123456".
They didn't decide to share the digits around to get more matches, e.g. "12345" + "678".
The greedy nature meant "123456" got matched and consumed for one match, leaving just "78" for the next.

For "123456789" though, the first match grabbed "123456" and there was enough leftover for a second match.

## Exercise 3 - lazy lower limit

We'll repeat the previous exercise but make the range lazy:

- highlight the area as usual
- do `/\v%V` as usual
- add `\d{3,6}` like before
- put a `-` before the `3` to make it lazy
- change the range to an exact one `{3}` - does it make any difference?

```
012     123456      0

00      12345678   0x23456

9987    123456789
```

For this example the lazy range `{-3,6}` was identical to just `{3}`,

ie. "3-6 matching as few as possible" is the same as "exactly 3".

The moment the lazy pattern finds 3 matches it yields, which is the same as `{3}`.

So are lazy quantifiers useful when you have lower bounds?
Are they always equivalent to an exact match using the lower bound?

There's a lengthy discussion about when lazy patterns are useful [here](too_lazy_for_laziness.md).

## Exercise 4 - upper limits

Select the "Boban"s with trailing digits, and in those cases match up to 5 digits.

- highlight the area as usual
- do `\v%V` as usual
- add `\d{}` (like before, matches should appear everywhere)
- put `,5` between the braces (the matches will change but it will be hard to see)
- put `Boban` before the `\d` (now the matches should be limited to Bobanish ones)

```
0123Boban    0Boban123         Boban123456789

Boban12345   Boban1234567890
```

The final pattern should be `\v%VBoban\d{,5}`

When we added the `,5` above, the core pattern became `\d{,5}` which means: 0 to 5 digits.
Like `\d*` or `\d{}`, this will have a zero width match on any non-digit.
The only difference adding `,5` causes at that point is that long strings like "1234567890"
would have split from one match into multiple matches.

## Exercise 5 - redemption, from lazy to greedy

In this exercise we'll show a common use of the laziness pattern, then flip it into a greedy pattern.

Write a pattern to match all the `<...>` words below. We'll be using very magic mode so we need to escape
the carets so that they're not understood as word boundaries.

- highlight the area as usual and do `/\v%V`
- add `\<`
- add `.*\>`
    - hmmm... this combines `"<a>between<b>"` into a single match when we wanted `<a>` and `<b>` separately
- replace the `*` with `{-}` (ie. make it lazy)
    - this fixes the bug above
    - but the bottom one is matching: `"<<hmm...>` (we'll deal with that in the next exercise)

```
<a>between<b>

<lonely open...

lonely close...>

<<hmm...> nested <carets>>
```

The final pattern should be:

```
\v%V\<.{-}\>                 \<       literal <
                             .{-}     0 or more anything (lazy)
\v  %V   \<  .{-}  \>        \>       literal >
```

At that point our core pattern was `\<.{-}\>` and for non-nested cases it worked well.

The `{-}` introduced is the lazy equivalent of `*` - it means:

> 0 or more of anything matching as few as possible (then a '>')

People often find lazy patterns confusing. We can change this to a greedy pattern:

> 0 or more characters that aren't '>' (then an '>')

So `.{-}` becomes `[^>]*`

Aside: In [the article](too_lazy_for_laziness.md), it talks about how you can change:

```
# Lazy version
[ATOM 1][LAZY QUANTIFIER][ATOM 2]

# Greedy version
[ATOM 1 and not ATOM 2][EQUIVALENT GREEDY QUANTIFIER][ATOM 2]
```

In our case ATOM 1 is `.` and atom 2 is `\>`.
Hence "ATOM 1 and not ATOM 2" means "any character except '>'" which we can represent with `[^>]`
(no backslash needed inside a character class).

Our equivalent greedy quantifier is `{}` or `*` (we'll use `*`).

So the overall replacement is:

```
# Lazy
/\v  %V  \<  .{-}   \>

# Greedy
/\v  %V  \<  [^>]*  \>
```

Highlight the below and try it: `/\v%V\<[^>]*\>`:

```
<a>between<b>

<lonely open...

lonely close...>

<<hmm...> nested <carets>>
```

You should have got the same results.

Overall the point of this exercise is to reinforce that you can often convert lazy quantifiers to greedy ones
by modifying the atom they're quantifying.
If people are less familiar with lazy quantifiers, this approach would make your patterns more readable.

## Exercise 6 - nesting

Carrying on exercise 5, we'll deal with the pesky nested case.

Generally speaking regex doesn't handle recursive structures.
There's a hilarious [stack overflow answer](https://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags)
which emphasizes this point better than I can.

For this example though, if we limit ourselves to just finding the innermost `<...>` chunks then we can still solve this.
The trick is to ban `<` characters "inside" the carets.

So we're changing it to:

- consume a `<`
- either:
    - lazily consume anything that isn't `<` (previously was just `.`)
    - greedily consume anything that isn't `<` or `>` (previously was just not `>`)
- consume a `>`

Comparing the before and after regexes:

```
# Lazy
/\v  %V  \<  .{-}     \>    Exercise 5
/\v  %V  \<  [^<]{-}  \>    Exercise 6

# Greedy
/\v  %V  \<  [^>]*    \>    Exercise 5
/\v  %V  \<  [^<>]*   \>    Exercise 6
```

Try out each pattern on the text below:

```
<a>between<b>

<lonely open...

lonely close...>

<<hmm...> nested <carets>>
```

## (Optional) Exercise 7 - reviewing the article - when is laziness useful?

(Note if you haven't read the [laziness article](too_lazy_for_laziness.md) then this exercise probably won't make sense)

Below we'll try a few patterns related to the article.
For each pattern we're testing whether the lazy quantifier behaves differently to the eager one.

Highlight the area below as usual and run each pattern below after the usual `/\v%V` in both lazy and eager mode,
e.g. for (a), run `a{-,5}` and `a{,5}`.

- (a) `a{-,5}`
- (b) `a{-,5}b`
- (c) `a{-,5}[ab]`
- (d) `[ab]{-,5}b`

```
aaabab  cab

a       da

b       ab
```

For (a), the matches were the same because there is nothing after the quantifier.
Without something after it, there's nothing to seize control off the lazy pattern, so it's effectively greedy.
This relates to the first condition from the article.

For (b), the matches are again the same. This is because the atoms `a` and `b` don't have any common charaters they match.
The lazy quantifier will never yield control to `b`, hence it's acting just like a greedy quantifier.
This relates to the second condition from the article.

For (c), the matches are different.
The greedy one will match long strings, whereas the lazy one has no effect on the pattern and you might as well simplify
the pattern to just `[ab]`.
In the language of the article, this is because B contains A, ie. `[ab]` matches everything `a` does,
hence it will always seize control off it immediately and the lazy quantifier will never match anything.
So whilst they behave differently, the lazy version is still useless.

For (d), the lazy and greedy matches are different _and_ the lazy pattern contributes to the match.
Looking specifically at the "aaabab" word:

- the lazy version will match "aaab" and "ab"
- the greedy version will match "aaabab"

In the language of the article this is because A contains elements B doesn't (ie. B doesn't contain A).
This comes from `[ab]` matching two characters 'a' and 'b', and `b` only matching 'b'.
Hence a string constructed from 'a's, then a 'b', then optionally more 'a's, and another 'b' will differentiate them.

# Conclusion

Vim's regex range syntax is very powerful for logic like: "at least", "at most", "from a-b" and "exactly a".

Ranges can be greedy or lazy, although in a lot of cases the lazy version is effectively the same as the greedy one,
or doesn't do anything at all.

Further in many cases where a lazy quantifier makes sense, you can modify the pattern to use a greedy one.

So you can get by using greedy quantifiers in a lot of situations, which might increase the readability
of your patterns as a lot of developers aren't so familiar with lazy quantifiers, or vim's unusual syntax for them.

# Appendix: Greedy vs Lazy

## Greedy

Greedy is what we're implicitly used to.

A pattern like `.*` is _greedy_ in the sense that it will consume as much as possible from the string in
its search for a match before yielding control to the next atom in the regex.

Highlight the below as usual, then do `/\v%V.*Boban<enter>`.

```
------------Boban----------Boban-------Boban------Boba----
```

If you do `gn` you'll notice it highlighted the entire match.

The crux of the pattern is `.*Boban` meaning: any number of anything, then "Boban"

There are a few substrings from above that would satisfy this from the start of the string:

```
------------Boban----------Boban-------Boban------Boba----
^^^^^^^^^^^^^^^^^
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Because `*` is greedy, the longest match ended up being the one found by the engine.

Below is a representation of the steps the engine goes through.
Whilst processing `.*` it powers all the way past the three "Boban"s because every character
is a match for `.`

Because it's greedy, the engine is just focused on the `.` atom.
When we do the equivalent laziness example, we'll see that at each 'B' the engine behaves differently.

```
---Bob------Boban----------Boban-------Boban------Boba----

-->-->-->-->-->-->-->-->-->-->-->-->-->-->-->-->-->-->-->- oh, hit the end, can't match "Boban"
              .* all the way baby!                B<---<-- need to backtrack
                                                  "Boba" hmm, doesn't match `Boban`
                                       B<--<--<--< keep backtracking
                                       Boban, matches `Boban` hoorah!
                                            -->--> start searching for the next match (none found)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
          FINAL AND ONLY MATCH
```

Once the `.*` hits the end of the string, the engine realizes the folly of its greed and begins to backtrack
until it finds something that matches the next atom `B`.

It eventually backtracks to the 'B' in "Boba" and tries matching the rest of the pattern (which fails).
So it continues backtracking until hitting the last "Boban" at which point it finds a complete match.
That match has consumed all the text up to and including the last complete "Boban".

With that match locked in, it will start searching for other matches in the remaining string but will find nothing.

Overall there's one big match from the start of the string up to and including the last "Boban".

## Lazy

A couple of observations about greediness:

- maybe you wanted to find the first "Boban", not the last
- maybe you wanted to find multiple matches, not just one super match
  (will make sense after the example below)
- if the string was really long and the last "Boban" is near the front,
  the regex engine will end up walking all the way to the end,
  then backtracking all the way back to the first match.
  Then it will do another journey to the end trying to find the next match.

Let's look at the equivalent lazy form of the pattern above.

Highlight the below as usual and do `/\v%V.{-}Boban<enter>`.

```
---Bob------Boban----------Boban-------Boban------Boba----
```

Hit `n` a few times and you'll realize that it's actually found 3 matches.

```
---Bob------Boban----------Boban-------Boban------Boba----
^^^^^^^^^^^^^^^^^
    match 1      ^^^^^^^^^^^^^^^
                   match 2      ^^^^^^^^^^^^
                                   match 3
```

This time the crux of the pattern is `.{-}Boban`.
Don't worry yet how the strange `{-}` syntax works, just trust that it means "0 or more lazily".
Hence `.{-}` means: 0 or more of anything lazily

This time the search engine starts at the beginning of the string and is walking forwards tentatively.
It's processing `.{-}` lazily which means that at each point it's looking at the next atom 'B' to see if
it can take over.

```
---Bob------Boban----------Boban-------Boban------Boba----

-->B                        stop processing .{-}
   Bob                      oh doesn't match `Boban`
    <-                      backtrack and process 'B' as part of .{-}
   -->-->-->B               process .{-} until hitting another 'B'
    .{-}    Boban           matches `Boban` hoorah! First match found
                 -->--> ... start searching for the next match

^^^^^^^^^^^^^^^^^
   FIRST MATCH
```

Whilst it's processing hyphens, it sees that they don't match `B`, so it continues processing `.{-}`.

When it encounters the first 'B' of "Bob" the lazy `.{-}` yields and the engine tries to match `Boban`.
It can't complete the match so it backtracks to the 'B' and continues processing after it with `.{-}`.

In the same way it keeps going until it hits the next `B`. Again it yields and tries to match `Boban`,
which it can, so it finishes the match.

Hence the first match spans the start of the string up to and including the _first_ "Boban".

It continues from the next hyphen trying to find more matches and finds two more.

## Comparing the two

For this specific example:

```
--------------------------------------------------------
         |      Number of     |     Amount of string   |
         |      matches       |       traversed        |
         |                    |      (worst case)      |
--------------------------------------------------------
         |                    |                        |
 Greedy  |    1 epic match    |      3 traversals      |
         |                    |                        |
--------------------------------------------------------
         |                    |                        |
 Lazy    |    3 small matches |      1 traversal       |
         |                    |                        |
--------------------------------------------------------
```

# Regex range syntax

The above example was a case of "0 or more".

Vim has a more generalized language for describing ranges:

```
# Complete ranges

Greedy: [ATOM]{a,b}

Match the atom a-b times, matching as many as possible.
e.g. x{3,5}    match 3-5 'x' characters, matching as many as possible.


Lazy:  [ATOM]{-a,b}

Match the atom a-b times, matching as many as possible.
e.g. \s{-2,6}  match 2-6 whitespace characters, matching as few as possible

ie. `-` means "lazy" and no `-` means eager.

---

# Partial ranges

When `a` is omitted, the effect is no lower bound (equivalent to a=0).
When `b` isn't specified, there is no upper bound.

e.g. \a{,6}    match up to 6 letters, matching as many as possible
     \d{3,}    match 3 or more digits, matching as many as possible
     \w{-,3}   match up to 3 wordy characters, matching as few as possible
     \k{-4,}   match 4 or more keyword characters, matching as few as possible

---

# No range

If both `a` and `b` are missing, then you can omit the comma.

There's 2 possible cases for this (greedy and lazy):

e.g. \w{}       match 0 or more wordy characters, matching as many as possible
     \w{-}      match 0 or more wordy characters, matching as few as possible

Note that {} is equivalent to *.

---

# Exact range

If you want to specify exactly n occurrences, then use {n} (no comma).

e.g. \i{4}     match exactly 4 identifiers

There's no concept of greediness or laziness here as it's exact.

---

# Magic

In very magic mode, you can use {...} without escaping.

In magic mode and lower they need to be escaped: \{...\}
however vim allows you to optionally not escape the closing brace.
```

So now we understand the `.{-}` from the previous example.

- `.` just means "any character" as usual
- `{-}` is range syntax
    - the `-` means it's lazy
    - there's no left part of the range which is like: "at least 0"
    - there's no right part of the range, so there's no upper limit

The effect is: 0 or more of any character lazily.

So just think of `{-}` as the lazy analog to `*`/`{}`.

# Regex flavors

You might be used to seeing the laziness represented as adding a `?` after a quantifier, e.g. `*?` or `+?`.

This doesn't work in vim. However in defence of vim, the `{a,b}` form they created is more powerful and generalized.

This is another example of divergence between regex flavors.
