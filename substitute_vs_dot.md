# substitute vs dot

## Manually replacing 

Imagine you're in a situation where you want to replace all the "boban"s with "enxhell"s below:

```scala
val greeting = "Welcome boban"
val bobanId = readId()
if (bobanMatches(name)) {
  ...
}
...
val farewell = "That will be all, thank you boban"
```

Some instances are in quoted text (the intended target), but some are in programmatic names (like `bobanMatches`)
and we'd need to be more careful about those.

It seems like we need to check through each match.

Thankfully `substitute` has the `c` flag for that, so we could do:

```
[RANGE] s/boban/enxhell/gc
```

## An alternative

What if instead we:

- ran a search: `/boban<enter>`
- used `n` to toggle through the matches
- on the first one we wanted to change, did `ciwenxhell<escape>`
- carried on using `n`
- used `.` to repeat that change on the matches we want to replace

In terms of key presses this is roughly the same.

# Comparing the two

For this case, what are the differences between the two?

## Simplicity

Using `.` and `n` doesn't require using a new concept (`substitute`).
It's just a clever combination of:

- buffer search
- operator + text object
- the dot operator

## Undo history

Using `.` with `n` makes each individual replacement a separate change from the perspective of `u` for undo,
whereas `substitute` makes it all one combined change.

Depending on your context that might be good or bad.

Certainly if you stuffed up the replacement, it's nice to be able to undo the whole thing in one go.

## Complex changes

As we'll see with later examples, the `.` with `n` pattern allows you to leverage the full power of normal mode.
For example you might want to:

- uppercase each match: "Boban" -> "BOBAN"
- toggle the case: "boBaN" -> "BObAn"
- increment a number: "40" -> "41"
- use the surround plugin: "boban" -> "(boban)"  (see katas 25 and 26)

The `substitute` command has limited power in how it can replace things.
It _can_ do many of the examples listed above, but it requires learning the complex substitution language
to achieve something we already know how to do.

## Matching complete text

With `.` and `n`, you don't need to match all of the text you need to modify.
You just need to match enough to get you close enough to make the change you want.

For example, suppose our task was: "delete the first set of (...) on each line starting with '#'"

```
# Comment (I don't like this)
some code...
# Comment, do something (this is no good)
some code...
# Comment (delete me please!) 
```

With `.` and `n`, all we need to do is search `^#` (lines starting with '#'), then do `dan)` for the first change,
and `.` on subsequent lines.
(This is assuming you've got targets.vim installed - see kata 35)

To do with this a `substitute`, we need something like:

```
[RANGE] s/\v^#.{-}\zs\(.{-}\)\ze//g
```

(things like `.{-}` and `\zs`, `\ze` will be covered in the advanced regex kata)
 
This is very complex because it requires the text we plan to change to be matched
(and not the text we _don't_ want to change).

The `.` and `n` approach can reuse the smarts from the targets.vim plugin to find that for us.

Overall `.` and `n` lets us leverage more of our familiar toolbox, and that toolbox grows as we add more plugins.
`substitute` though has a baked in fixed set of tricks we can use, and requires matching all of the text we want to change.
Learning those tricks is extra cognitive load and because we don't use them as much as normal mode tricks,
they tend to get forgotten.

## False positives

With both approaches, because you're interactively checking each match, it's okay to use a simpler search term
that might let in a few matches you don't want to change (ie. false positives).

Often this saves you having to create complex regexes to handle curly edge cases.
Often the time it takes you to dust off your advanced regex knowledge and get it working is longer than the
time it would take to just manually skip over the false positives.

# So when is substitute useful then?

Here's a couple of cases that stand out to me:

## Scale

If you're confident that you don't need to preview each change, then the substitute approach works much better at scale
when you have many replacements to do.

It's also much easier to express in automated scripts (some examples in kata's 40 and 49) and will perform better.

## Leveraging the internal structure of a match

Replace logic can use groups and back references cleverly to transform text.
There isn't a normal mode analog to that.

For example this pattern switches the first 3 digits with the second 3, e.g. "123456" -> "456123":

```
RANGE s/\v(\d{3})(\d{3})/\2\1g
```

(See kata 50)

There isn't a simple normal mode command to capture that kind of change easily.

You _could_ do it with a macro though: `qqd3lllpq` (see kata 21)

- `qq` record into register `q`
- `d3l` delete 3 characters
- `ll` move right 2 characters
- `p` paste after the cursor
- `q` stop recording

Then the `.` and `n` pattern changes to `@q` and `n` which has the same spirit.
You do `/\v\d{6}` to find your matches then `@q` on each one you want to replace.

# The e flag

Buffer search `/` has a useful search flag `e` for putting your cursor at the _end_ of each match.

For example suppose we were wanting to change bobany words like "Boban", "boban", "bobn"
into their feminine form by adding "ita" to the end.

You could do do this with a substitute command:

```
[RANGE] s/\v\cboba?n/\0ita/gc
```

or you could do:

```
/\v\cboba?n/e<enter>
```

then on your first change, do `aita<escape>`:

- `a` go into insert mode after the cursor
- `ita` type in "ita"
- `<escape>` return to normal mode

Then `n` and `.` as usual.

```
boban said to bobn,
who's that new Boban over there?
```

In the above example, we could have not used `e` and jumped to the end of the word with `e` from normal mode, ie.

```
# Search
/\v\cboba?n/

# First match...
# Jump to end of word and insert "ita"
eaita

# Next match
e.
```

It does mean though that we have to keep hitting `e` before each application of `.`.

Imagine a more complex example like matching "boban" or "boban.jones".
Hitting `e` will get us to the end of "bobans", but not "boban.jones" (because of the '.').

So adding `e` will make it easier for you to create dot repeatable patterns oriented to the back of your matches
and really shows its value when patterns get more complex.

# Other alternatives

This article has been about comparing `substitute` with `.` and `n`.

I covered these because they usually overlap the most in terms of use cases.
But there are other ways to skin a cat too. Like:

- the global command (kata 29)
- the `Subvert` command (kata 52)
- a well made plugin (if what you're doing is fairly common)

# Summary

Do we really need the `substitute` command?

I think the answer is: probably not as much as you think.

By that I mean your brain might have a natural tendency to think "find and replace!" for certain tasks
because that's what you'd in other editors, but often vim will have superior strategies that those other
editors didn't have.

Getting `substitute` to the same power as actions you can already do from normal mode requires learning a lot of tricks
and adhoc techniques that you'll tend to forget because you won't use them regularly.

When creating these kata, I wanted to give more time for developing habits around simpler powerful tools like
operator + text object and the dot operator.
So I've delayed introducing `substitute` until kata 14,
and left the really advanced tricks all the way to katas 50 and 51!

Having said all that, `substitute` is definitely the best tool in some situations.
For examples scripts/automation tasks and replacements at scale.
