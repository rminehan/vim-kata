# Too lazy for laziness?

This article relates to [kata 48](kata 048_advanced_regex_6_ranges) which introduced lazy quantifiers.

The question we're interested in answering is: When are lazy quantifiers useful?

Sitting behind this question is an assumption that most developers are much more familiar with greedy
quantifiers from using `+` and `*`, so we want to avoid introducing a lesser known concept into our patterns
unless it's useful.

For this article I'm defining "useful" as:

- the lazy quantifier causes different matching behavior to the greedy quantifier
- the lazy quantifier can't be swapped for a simpler non-lazy quantifier

# Quick reminder

Remember the `*` quantifier is the same as `{}`.

We'll use `{}` throughout to mirror its lazy analog `{-}`.

# Set terminology

Throughout this article we're going to be talking a lot about the set of characters an atom matches.

For example:

- `a` matches 'a'
- `.` matches every character (except newlines)
- `[^b]` matches every character that's no a 'b'

(we'll be assuming case sensitivity throughout to keep things simple)

We'll use `~[ATOM]` to represent the characters of values matched by an atom, e.g.

- `~a` is just 'a'. 
- `~[^b]` is the set of characters that aren't 'b' (and whitespace)

# When do lazy and greedy behave differently

## Motivating Example

Consider a pattern like `a{-}` (0 or more 'a's lazily) and its greedy equivalent `a{}`.

Is there any situation where they match differently?

Try them both below and you'll see they match the same things.

```
aaaab     b    aaaa    baaaa
```

Now we'll add a second atom `b` after the quantifiers: `a{-}b` and `a{}b`.

Try them both and you'll see that they both match identically.

Not only are they identical, but in this case, the lazy one will perform worse in situations of long 'a' chains,
because it's checking every 'a' for whether it matches `a` _and_ `b`.

Our goal for this section is to discover cases where laziness causes different behavior and so becomes "useful".

## The conditions

For laziness and greediness to differentiate, you need two conditions:

- the quantified atom must have an atom after it
- the subsequent atom must match something the first atom would match

### Point 1

> the quantified atom must have an atom after it

This is saying that if there's nothing after the lazy atom,
then the laziness never has a chance to affect the behavior by jumping
the engine to the next atom.

We saw this with the `a{-}` example.

### Point 2

> the subsequent atom must match something the first atom would match

This is saying that for there to be a meaningful difference between laziness and greediness,
you need a situation where _both_ atoms would match a character.

In that situation:

- the greedy quantifier will match it
- the lazy quantifier will yield to the next atom which matches it

In the pattern `.{-}x`, there _is_ a character that both atoms match: 'x'.

Hence the string "--x--x" causes different matching behavior between `.{-}x` and `.{}x`.

In our second motivating example, we added `b` after the quanified atom.
However `a` and `b` don't have any characters they both match,
so lazy and greedy versions still behave the same.

Another way to express this condition is that `~[ATOM 1] intersect ~[ATOM 2]` must be non-empty.

`~a` and `~b` are just the sets 'a' and 'b' respectively and don't intersect.

But `~.` and `~x` do intersect on 'x'.

### Point 2 - expressed algorithmically

> the subsequent atom must match something the first atom would match

Here's another way of looking at it - the regex engine's logic for matching with quantifiers
would be something like this:

```
if current atom matches current character AND number of matches satisfies quantifier
  if quantifier is lazy AND next atom matches current character
    consume it as part of the _next_ atom   // ***
  else // quantifier is greedy
    consume it and stay on current atom/quantifier // %%%
else
  // Move to next atom
  ...
```

For a lazy quantifier to cause different execution to the equivalent greedy one,
we need execution to get to the `***` line at least once.

The only way that will happen is if the two `if` statments before both pass.

That would require a character to exist such that the currenet atom and next atom both match it.

# A and B

For the remainder of this article, we'll use A and B like so:

```
In a pattern like [ATOM 1][LAZY QUANTIFIER][ATOM 2],

A is ~[ATOM 1]

B is ~[ATOM 2]
```

e.g. in `\d{-}k`, A is all digits and B is 'k'

# A general form for the text matched

## So far:

We've got two rules so far that are necessary for lazy and greedy quantifiers to behave differently:

- there must be an atom after the quantifier
- A intersect B must be non-empty

## A general form for the text matched

We have minimum rules to make the lazy and greedy matches different.

These are necessary but not sufficient.

We also need the text being matched to allow the cases to diverge which needs text like:

```
wwwwwwxyyyz                 w's are in A but not B
      |   |                 x   is in A and B
^^^^^^^   |   lazy match    y's are in A and are optional
          |                 z   is in B
^^^^^^^^^^^   greedy match
```

An example would be for the pattern `.{-}\d`:

```
abcdef0ghi1
      |   |
^^^^^^^   |   lazy match
          |
^^^^^^^^^^^   greedy match
```

Note the symbols above like w and y aren't necessarily all the same characters as each other (as in the example).
They just represent any character satisfying a set based condition.

The lazy version stops on the x because it's in B.

The greedy version powers through the w's and x because they're all in A.
To make it different to the lazy case we can't have it stop on x though.
So we need another element in B further in (marked by z).

z could be right after b or have a chain of characters from A in between for the greedy
quantifier to consume whilst it looks for that final element in B.
That chain is represented by y's. They may or may not be in B.

z itself doesn't need to be in A, it's main job is to be in B.
It it is in A, the greedy quantifier would have consumed it,
then either hit the end of the string or found the next character to _not_ be in B.
It would have then backtracked to z and matched it as part of B.
If z isn't in A, that implies that the greedy match ran out just before it.

## Constructing examples

The `wwwwwwxyyyz` form above gives us a way to construct examples where lazy and greedy quantifiers differ.

Just find two atoms, determine the A and B sets, then work out values allowed in the place holders based on the rules.

# Simplifying patterns

The intro defined "useful" as:

- the lazy quantifier causes different matching behavior to the greedy quantifier
- the lazy quantifier can't be swapped for a simpler non-lazy quantifier

The sections above have fleshed out what is required to make to satisfy the first dot point.

This section will focus on the second.

## Motivating examples

The examples below will show that A intersecting B isn't enough to make the lazy quantifier useful.

In each example A intersects B _too_ much...

### Example 1

Suppose we construct a lazy pattern using atoms `a` and `[ab]`, e.g.

`a{-,4}[ab]`: at most four 'a's matching as few as possible, then a or b

Let's look at its behavior below:

```
aaab   b    aaaaaaaab
```

Now search just `[ab]` in the same way and you'll find the matches are the same.

The root cause is that `[ab]` matches everything `a` does, so the `[ab]` always seizes control
from `a` as it's using a lazy quantifier.

More generally, the issues is that B contains A. Here A is 'a' and B is 'a' and 'b'.

So for this example, we could have just simplified the pattern to `[ab]` as the lazy quantifier never consumes anything.

### Example 2

Let's change the pattern to have a minimum bound:

`a{-2,4}[ab]`: two to four 'a's matching as few as possible, then a or b

Try it below:

```
aaab   b    aaaaaaaab
```

Here the `a{-2,4}` part only ever matches 2 'a's, because whenever there's a third 'a', it will yield to `[ab]`.

For this example, we could have just simplified the lazy quantifier to `{2}`, ie. `a{2}[ab]`.
Search again with this pattern to confirm for yourself.

Again this stems from B containing A.

### Example 3

Just to reinforce this, let's look at a lazy range with a non-zero lower bound.

Search `a{-3,}[ab]`: 3 or more a's, then an a or b

```
aaab   b    aaaaaaaab
```

Now search with `a{3}[ab]` and you should find they're the same.

The lazy quantifier only ever matches 3 'a's, because any fourth 'a' it encounters would be stolen by the `[ab]`.

## The minimum required

In a situation where B contains A, the lazy quantifier will _always_ do the absolute minimum it has to.

This means we can simplify the pattern.

In the first example, the minimum for `a{-,4}` is zero, so it simplifies to `a{0}` and we can just omit it.

In the second example, the minimum for `a{-2,4}` is two, so it simplifies to `a{2}`.

In the third example, the minimum for `a{-3,}` is three, so it simplifies to `a{3}`.

In all cases, the simplification means that the lazy quantifier is replaced by an exact quantifier or just disappears completely.

The point being made here is:

> If B contains A, lazy quantifiers are not useful.
> They can be replaced by a simpler non-lazy exact quantifier or omitted completely if the quantifier is 0.

# Summing up so far

Looking back we've now got a bunch of necessary conditions to make lazy quantifiers useful:

- must be followed by an atom
- A and B must intersect
- there must be text being matched in the form `wwwwwwxyyyz` above
- B can't contain A

There's 4 relationships between A and B:

- (a) A equals B
- (b) A strictly contains B
- (c) B strictly contains A
- (d) A and B don't strictly contain each other

A "strictly" contains B above means A has everything B has, and more.

If B can't contain A, that knocks out (a) and (c) leaving (b) and (d).

Let's look at some examples:

## A strictly contains B

This is the most common case.

Mostly if two primitive atoms intersect, one will completely contain the other. For example:

- `.` and `x` - 'x' is strictly contained in the set of all characters (except newline)
- `\w` and `\d` - all digits are words, not just some
- `\a` and `[a-z]` - all lower case letters are letters, not just some

## A and B don't strictly contain each other

We can manually construct atoms like this, e.g. `[abX]` and `[abY]`.

In terms of primitive atoms, `\k` and `\i` as defined in [kata 46](046_advanced_regex_4_keywords_identifiers_files.md)
happen to be examples of this:

```vim
" A~\k = English alphabetical characters, underscore, asterix
set iskeyword=a-z,A-Z,_,*

" B~\i = ascii letters (English, German, Italian etc), digits, underscore
set isident=@,48-57,_
```

Keywords and identifiers intersect on English letters and underscore.

Keywords have the '*' character which identifiers don't.

Identifiers have the non-English letters which keywords don't.

So we could use either as the first or second atom and come up with some useful lazy quantifiers.

For example: `\k{-}\i` will behave differently to `\k{}\i` on "***ÖabcÖ".

In the particular example of keywords and identifiers though,
it's unlikely we'd ever be searching a mixed regex like this though.

# Transforming patterns

Let's look again at:

> the lazy quantifier can't be swapped for a simpler non-lazy quantifier

## Warm up example

Our warmup example from the kata was searching for "Boban"s in:

```
------------Boban----------Boban-------Boban------Boba----
```

we showed that `.{-}Boban` and `.{}Boban` behaved differently.

In the terms of this article, we're only interested in the atom directly after the quantifier (`B`)
as it's the one the lazy quantifier considers.
So really we're talking about `.{-}B`

This example passes all our tests:

- the lazy quantifier must be followed by an atom (yep there's `B`)
- A and B must intersect (yep they intersect on 'B')
- there must be text searched in the form `wwwwwwxyyyz` (yep the example above)
- B can't contain A (yep B is just the character 'B', A is everything (except newline))

Even here though, we can still get rid of the lazy quantifier.

Search this pattern on the block above: `[^B]{}Boban`.
Like our lazy pattern `.{-}Boban` it finds 3 matches, and it's using a greedy quantifier `{}`.

It means: match 0 or more not-B's greedily, then match Boban

## Generalizing this

What we did here was change from:

```
# Old
[ATOM 1][LAZY QUANTIFIER][ATOM 2]

# New
[ATOM 3][GREEDY QUANTIFIER][ATOM 3]

where ~[ATOM 3] equals ~[ATOM 1] subtract ~[ATOM 2]
```

In this language:

- atom 1 is `.`, and `~.` is all characters (except newline)
- atom 2 is `B`, and `~B` is just 'B'
- atom 3 is `[^B]`, and `~[^B]` is all characters except 'B' (and newline)

The original quantifier was `{-}` and the equivalent greedy quantifier was `{}`.

## Another example

If we said "find all zeroes with up to 6 non-zero digits before them", we could do:

```
# Lazy
\d{-,6}0

# Greedy
[1-9]{,6}0
```

Here `\d` subtract `0` is the same as `[1-9]`.

Try it out below:

```
1234560     0         0123

000      76543210
```

This is an example where the greedy version happens to be very simple to generate and is
a more intuitive representation of the task description, particularly the part: "up to 6 non-zero digits".

## Can/should we always do this?

Looking back at the transformation:

```
# Old
[ATOM 1][LAZY QUANTIFIER][ATOM 2]

# New
[ATOM 3][GREEDY QUANTIFIER][ATOM 3]

where ~[ATOM 3] equals ~[ATOM 1] subtract ~[ATOM 2]
```

The crucial thing was being able to find that atom 3 whose set of characters happens to match atom 1's subtract atom 2's.

We can always construct one by with an `&` and a negative look ahead, e.g. `(atom 2)@! & (atom 1)`.

This literally means "not atom 2 AND atom 1". Then we need to wrap that in a quantifier and add atom 2.

Using it on our previous example we'd get: `((0)@!&\d){}0` below:

```
1234560     0         0123

000      76543210
```

So we _can_ always do this. The question is whether we _should_.

The pattern above was very complex and in order to get of laziness, we introduced other concepts that are unfamiliar
to most developers.

So the ideal situation is when there happens to be a simple atom that happens to capture A minus B.
For example how `[1-9]` elegantly captures "digits that aren't 0".

# Summary

When are lazy quantifiers useful? ie. when do they cause different behavior to their greedy counterpart
and can't be replaced with a simpler greedy expression?

We're asking this because we want to avoid introducing lesser known concepts in our regex if there's an
equally good couterpart people already know.

We've seen that it actually requires fairly specific situations to justify a lazy quantifier:

- the lazy quantifier must be followed by an atom
- A and B must intersect
- there must be text searched in the form `wwwwwwxyyyz`
- B can't contain A
- there isn't a simple enough atom that captures "A minus B" to justify replacing converting to a greedy pattern

So sometimes we're justified in being too lazy to be lazy.
