# Why avoid newline?

[Kata 49](049_advanced_regex_7_multiline.md) introduced multiline regexes and explained that by default,
atoms like `.` and `\s` _don't_ match newline characters.

What is the motivation for making this seemingly ad-hoc exclusion?

This article addresses this question with the crux of the argument being that developers tend to think in a "line oriented"
way and those operations are much simpler to perform when newline is excluded from common atoms.

To make the point, we've got some examples below which show what it would be like to solve common problems
if atoms _did_ match newline.
A lot of bugs quickly arise and working around them is hard or impossible without just excluding newline.

# Example 1

Suppose we wanted to find all the commented out text below, but not the "#" characters themselves or any whitespace
between '#' and the start of the comment.

```
#
# Commented
# Also Commented
#
Not commented
```

We might search `\v%V#\s*\zs.+\ze`

```
\v  %V  #  \s*  \zs  .+  \ze         #    find a hash
                                     \s*  consume all the whitespace after it
                                     \zs  start matching
                                     .+   at least one character (we didn't use `*` because some lines are empty comments)
                                     \ze  end matching
```

This correctly matches "Commented" and "Also Commented" because they have at least 1 non-whitespace character
after the comment.

Now suppose that `.` and `\s` matched newline characters. This will mess up the match:

```
#\n# Commented\n# Also Commented\n#\nNot commented

   ------------------------------------------------
                one big match
```

The `#` matches against the first '#', then `\s` matches against '\n'.

From there the `.+` will match the remainder of the string as it can match newlines now.

The end result is one big match beginning from the second '#'. There's two problems with it:

- it's counting that second '#' as a "commented out" character
- it's rampaging to the end of the string creating one super match

## Fixing the rampage (first try)

You're probably thinking:

> put `^` and `$` into the pattern to fix the line boundaries.

```
# Old
\v  %V  \v     #  \s*  \zs  .+  \ze

# New
\v  %V  \v  ^  #  \s*  \zs  .+  \ze  $
```

This unfortuantely doesn't work though.
Putting a `$` at the end of the match doesn't put a leash on `.+`.

```
#\n# Commented\n# Also Commented\n#\nNot commented
   -----------------------------------------------
                                                  ^
                                                  $ matches here
```

The first '#' will match `^#` being at the start of a line, then `\s*` will consume the newline character,
leaving us at the second '#'.

From there `.+` takes over and consumes everything in its path.
It's not "looking ahead" to the `$` and knowing that it's supposed to stop.
It just consumes until the end of the text.

Then `$` matches _that_ newline and the match is complete.

So `^` and `$` didn't solve the problem of the runaway regex.

It becomes pretty clear that greedy patterns like `.*` would be mostly unusable for "line based" problems.

## Fixing the rampage (second try)

The reason `$` didn't help above was because `.+` wasn't looking ahead to it.

Hopefully this is tweaking your brain to think of lazy quantifiers.

ie. change `.+` to `.{-1,}` meaning "one or more of anything lazily"

```
# Old
\v  %V  \v  ^  #  \s*  \zs  .+      \ze  $

# New
\v  %V  \v  ^  #  \s*  \zs  .{-1,}  \ze  $
```

Indeed this does fix the rampaging issue.

## Fixing the hash logic

With the above we still have this annoying issue where if you have an empty comment line,
then the line after it gets understood as being commented out by it.

It comes from `\s` matching newline in our thought experiment.

Our current pattern is:

```
\v  %V  ^  #  \s*  \zs  .{-1,}  \ze  $
```

And it leads to these matches:

```
#\n# Commented\n# Also Commented\n#\nNot commented
   -----------    --------------     -------------
```

Like before, the first hash is matching `^#` and then the first newline is matching `\s*`.
That `\s*` was intended for whitespace after the `#` and before text on the same line - it was never meant for a newline.

Then "# Commented" gets matched by `.{-1,}` until a newline is hit.

The next match works well because it _doesn't_ have an empty comment line before it.

Then "Not commented" gets incorrectly matched due to the same bug as it has an empty comment line before it.

The simplest solution here is just to make `\s` _not_ match newline (ie. the default setting).

### A quick recap

Hopefully you're starting to see that the default settings of _not_ matching the newline character actually make sense.

To hammer it home we'll do one more example.

# Example 2

Suppose we were wanting to shrink multiple whitespace characters after "Boban" down to a single character.

```
# Before
Boban 		Jones often uses up too many spaces. That Boban
	is a real Boban   don't you think?

# After
Boban Jones often uses up too many spaces. That Boban
	is a real Boban don't you think?
```

We'd search for `\s+` following "Boban" and replace it with just a space. That translates to a `substitute` command:

```
%s/\vBoban\zs\s+\ze/ /g

%   s   /\vBoban\zs\s+\ze   /SPACE     /g
                             replace    global (more than one per line)
         \v
           Boban
                \zs         start matching
                   \s+      one or more whitespaces
                      \ze   end matching
```

If `\s` included the newline character then the matches we'd get would be:

```
Boban \t\tJones often uses up too many spaces. That Boban\n\tis a real Boban   don't you think?
     -----                                               ----               ---
      :)                                                  :(                 :)
```

The issue is the middle match - it's shouldn't be there.
But because "Boban" ends the line, there's a newline after it which gets matched by `\s`.
Even worse the line after starts with whitespace so that's getting included in the match too.

Both characters will get replaced with a space which will have the effect of joining the two lines into one.

```
Boban 		Jones often uses up too many spaces. That Boban
	is a real Boban   don't you think?
```

The easy fix is to just make `\s` ignore newlines.
Maybe you're stubborn though and want to persevere with `\s` matching newlines.

## Fixing it (the hard way)

We could "fix" the search by adding `[^\n]` or `[^\$]` after `Boban`, ie. "Boban's that aren't before a newline".

That will miss Boban's that have whitespace after them, _then_ a newline:

```
This Boban 
is        ^ trailing space
sneaky
```

So we change it to: Boban's that aren't followed by whitespace then the end of line

That's tricky to represent though when we're also trying to shrink that same whitespace and need to match it.

Our pattern would be:

```
\v%VBoban(\s{-}$)@!\zs\s+\ze

\v  %V  Boban  (\s{-}$)@!  \zs  \s+  \ze

               ^ negative lookahead
                 zero or more spaces (lazily) then end of line
```

```
This Boban 
is        ^ trailing space
sneaky
This one has no trailing space: Boban
This Boban    should get some trimming
```

This works in that it only matches the whitespace after the last Boban. It's quite complex though.

## Summing up this problem

The easiest fix is just to make `\s` ignore newlines.

This problem is another "line based":

> Suppose we were wanting to shrink multiple whitespace characters after "Boban" down to a single character.

"after Boban" here really means "after Boban on the current line".

# Final conclusion

The majority of problems we solve with regex in vim are "line based" so it makes for the default behavior
of atoms like `.` and `\s` to _not_ match newline.

This contains matches to a single line by letting newlines act as fence posts between matches.

For the cases that you _do_ want to cross multiple lines, there are `\_x` versions of these operators
that do that for you.
