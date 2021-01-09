# The dangers of visual mode

This is a click-baity title to get you thinking about visual mode and when to use it.

Overall the message is that visual mode is something you should try to avoid when possible
as it's not as ergonomic as alternatives or has very unexpected behavior.
Sometimes though you don't have a choice and visual mode is the easiest way to do things.

A lot of this material is already dispersed throughout the kata so I decided to collect it all into one article,
meaning there will be some duplication.

The starting segments of the article assume you're familiar with visual mode - see kata's 8 and 9 for reference.

Later sections cover more advanced topics and they'll mention the relevant kata in them.

Some of the sectios below might be based on bugs that later get fixed, or just improved. I'm using neovim v0.4.4.

# Reproducibility

This section is inspired by tip 23 from Practical Vim.

[Kata 1](001_dot_operator.md) introduced the dot operator and the idea of making changes in a way that's reproducible.

For example if we wanted to upper case the "Boban"s or "Bobanit"s below, we could put our cursor anywhere on one of the Boban's
and go `gUiw` (uppercase inside little word).

```
Boban is as Boban does
He's a big Boban
Bobanita isn't the same, that's what Bobanita said
```

After the first uppercase operation, we can use move our cursor to the other "Boban"s and "Bobanita"s and just press `.`.
For this discussion, it doesn't matter how we move the cursor around to each word, the important thing is that the change
itself was expressed in a reproducible way and once we got to each word, we can just use `.`.

It's a simple but very powerful pattern.

Let's do the same thing using visual mode.

- put your cursor to the 'b' on the first "Boban" below
- do `viw` to visually select it
- do `gU` to uppercase it
- move your cursor to the 'B' on the next "Boban"
- hit `.` (all good so far)
- move your cursor to the 'b' on the next "Boban"
- hit `.` and you'll see it just uppercases "ban" - whoops!
    - okay so we have to remember to put our cursor at the start of the word
- move your cursor to the 'B' on the first "Bobanita"
- hit `.` and you'll see it just uppercases "Boban" and not "ita" - whoops!

```
Boban is as Boban does
He's a big Boban
Bobanita isn't the same, that's what Bobanita said
```

What we did above was define an action that said: "uppercase the 5 letters from the cursor".

When we did `viw`, it created a visual selection of 5 characters in that context.
The initial location of the cursor on 'b' didn't matter.
Hitting `gU` created an action based on the area.

Visual mode isn't an intelligent "recipe" the way `iw` is from the first example.
It doesn't record that fact that we used `iw` to initially create it, it's just a fixed length block of characters
and that length happened to be 5 because the word we used to create the change had 5 characters.
If we'd started on "Bobanita" it would have been an 8 character block.

So use visual mode like this, we'd need to:

- make sure all our words are the same length (give or take some whitespace)
- always apply the change at the start of the word

Compare that to `gUiw` where:

- words can be different lengths, they just need to be "little"
- you can apply the change from any character you want

Both strategies have about the same number of keystrokes to create the first change,
so `gUiw` is a clear winner here.

Understanding the static way visual blocks are recorded will be important for later sections
and we'll see it can lead to unexpected behavior.

# Static visual areas

## Linewise example

Visual areas are handled a little inconsistently.

- put your cursor on the `11111` line
- do `V`
- do `jj`
- hit escape
- confirm your last visual selection with `gv`
- put your cursor on the `22222` line and do `dd`
- hit `gv` to see how the visual selection updates
    - it should now just span _2_ lines as expected
- hit `u`
- delete the `33333` line in the same way and hit `gv`
    - it should span _3_ lines ending on the `44444` line :sad-parrot:
- hit `u`
- delete the `11111` line and hit `gv`
    - it should span _2_ lines :hmmm-parrot:

```
00000
11111
22222
33333
44444
```

The point of the exercise is to show that there is some inconsistency in how visual areas adjust
when the text inside them changes.

## Playing around more

Play around more with the example:

- put text at the start or ends of lines
- add new text in the middle of the selection
- switch `11111` with `22222`
- switch `11111` with `00000`

```
00000
11111
22222
33333
44444
```

## Character example

Now a similar example in character visual mode:

- put your cursor on the '1' from line B
- do `v`
- do `jj`
- press escape
- do `gv` then escape to confirm the selection
- delete "B0"
- delete "D0"
- do `gv` then escape to see how the selection updated
- delete the whole C line
- do `gv` then escape to see how the selection updated

```
A0123456
B0123456
C0123456
D0123456
E0123456
```

Deleting "B0" and "D0" don't shift the selection. It now starts and ends at '3'.

But deleting the C line _does_ update the visual area.

This particular example will be quite relevant later on.

## When does this matter?

In practice this isn't a big deal because usually when you create a visual selection,
it's to immediately operate on the text then you forget about it.

However if you start using visual mode based logic inside scripts and plugins for more complex logic you might
get stung on some edge cases.

Likewise there _are_ some operations which rely on the behavior of `gv` whilst also modifying the text within the block.
We'll see an odd examples of this with `substitute` later.

# How to think of visual mode

Visual mode makes sense for constructing "adhoc" text objects generally to be used for one operation.

By "adhoc" I mean text areas that can't easily be described by built in text objects (e.g. word, line, paragraph),
or a motion based text object like (e.g. "until the next comma `t,`").

Because you're only doing the operation once, it doesn't matter that it's not a reproducible action.

For new vimmers, making this mental adjustment can be hard.
You might tend to think of visual mode as the analog highlighting in vanilla editors and in those contexts
it's often the best way to do things, so it's become a built in habit.
But in vim, there _are_ much better ways to do a lot of tasks than using visual mode.
In my experience, I don't need visual mode very often.
If you find yourself using it a lot, chances are you're leaning on it too much when better options are there.
Kick the habit!

# Times you have to use visual mode

Having just said to avoid visual mode, there are times however whenever you have to use it due to what I'd call
deficiencies or inconsistencies in vim.

For example with indentation (covered in [kata 34](034_indentation.md)), the indentation operator doesn't work
as expected with counts, and sometimes the easiest way to indent a block of lines multiple times is by visually
selecting it.

Another example is trying to represent line based text objects as ranges for ex commands.
For example suppose you wanted to run an ex command over the current paragraph.
Ex commands take ranges and don't provide a way to specify a text object like `ip` to define a range.
Instead you can do `vip` and let `:` convert that into the range `'<,'>` for you.
More on that next!

# Ranges from visual selections

## Quick Recap of marks and ex commands

Throughout our coverage of ex commands it's also come up that you can use the line number of marks in ranges.
Let's do an example to refresh our memories:

- put your cursor on "1"
- do `ma` to put an `a` mark there
- put your cursor on "3"
- do `mb` to put a `b` mark there
- do `:'a,'b normal Ax<enter>` to append an 'x' to the lines from mark `a` to mark `b`

```
0
1
2
3
4
```

The single quote because the marks in the ex command mean "line number of mark".
They're analogous to how in normal mode, a single quote followed by a mark takes you to the start of the line
that mark is on. Try it by doing `'a` then `'b`.

## Combining with visual mode marks

[Kata 28](028_built_in_marks.md) introduces the built in marks `<` and `>` which represent the start and end
anchors of the most recent visual selection.

We can use them in a similar way:

- put your cursor "1"
- do `V`
- press `jj` to extend it down to "3"
- press escape
- press `gv` then escape to confirm the last visual selection
- do `'<` to confirm the start of the most recent visual selection is at "1"
- do `'>` to confirm the start of the most recent visual selection is at "3"
- do `:'<,'> normal Ax<enter>`

```
0
1
2
3
4
```

## Auto-populating the range

Vim has a nice trick where if you're in visual mode and press `:`, it puts `'<,'>` in for you.

- put your cursor "1"
- do `V`
- press `jj` to extend it down to "3"
- do `:` and note how it's put the range in for you
- add `normal Ax<enter>`

```
0
1
2
3
4
```

Sometimes you don't want this range as you just _happened_ to be in visual mode when you hit `:`.

You can use `<c-u>` (covered in [kata 33](033_insert_tricks.md)) to delete to the front of the ex command.

# Limiting regex searches

[Kata 43](043_advanced_regex_1_restricting_search_space.md) introduces many mechanisms to restrict matches
in a search based on context within a buffer.

One such mechanism is `%V` (or `\%V` in magic mode) which only matches if the engine is inside the most
recently selected visual block (see `:help /\%V`).

It turns out there's some bugs using this with the `substitute` command.

## Kata 49 - exercises 1 and 2 - prison break

[Kata 49](049_advanced_regex_7_multiline.md) exercises 1 and 2 show the effect of including or not including
`%V` at the back of a pattern used in a substitute.

The aim of the exercise is to cleanup trailing whitespace and blank lines within a visually selected block.

```
# With %V
s/\v\s*$\_s*%V/ /

# Without %V
s/\v\s*$\_s*/ /

# Expanded search term
\v  \s*  $  \_s*  %V
```

Visually select just the lines from "Join me" to "and this one" and run the first substitutions above.

```

  Join me
    with the
  the lines
below

 and this one


```

You should get:

```

  Join me with the the lines below and this one

```

## That's not quite right though...

Notice how the original text had 2 blank lines after it, but the output one had 1.

This actually doesn't make sense based on our current understanding.
You might be thinking that the newline after "one" is being matched and replaced by space.
That's not the case though.

Let's track what the engine should do based on our understanding:

```
\n  Join me		\n    with the  	\n  the lines\nbelow\n\n and this one\n\n\n```
    -----------------------------------------------------------------------
    ^             Visually selected                                      $ matches
    ^                                                                    -->--> \_s*
    ^                                                                    --<--< backtrack for %V
    ^                                                                    %V matches the \n but doesn't consume it
    ^                                                                   ^
    --------------------------------------------------------------------- ends on 'e'
                          Expected final match
```

- at the newline after "one", `\s*` will have a zero width match
- `$` will match before the newline (zero width)
- `\_s*` will match the newline and run all the way to the 3 backticks
- the `%V` won't match at the first backtick as it's not in the original visual selection
- the engine will backtrack until the _following_ character is in the visual selection
    - that is _before_ the newline after "one"
- the last character consumed was 'e'

The engine should finish just before the newline after "one" and _should not have_ replaced it.
There _should_ be 3 newlines meaning two blank lines.

## The character _before_ the end

This comes back to a subtle aspect of `\%V`. From the help:

```
\%V     Match inside the Visual area.
        When Visual mode has already been stopped match in the area that gv would reselect.
        This is a zero-width match.
        To make sure the whole pattern is inside the Visual area put it at the start and just before the end of
        the pattern
```

Note the part about _before the end of the pattern_.

In our pattern, the `%V` is after the end of the pattern in the sense that it's testing if the next element
after what's been consumed so far is in the visual selection.

For example if you selected the below as usual, then searched: `/\vabc%V` it will be checking if the 'd' was
visually selected, not 'c'. This is because the engine has already consumed 'c' using `c` and its position is
now at 'd' when it processes `%V`.

```
abcde
```

The 'd' would be checked aginst `%V` but it's a zero width match, so the 'd' wouldn't be consumed.

If you were doing a substitution, the 'd' wouldn't be touched.
Try it by visually selecting the line and doing: `:s/\vabc%V/x<enter>`

If you wanted to make sure that only "abc" needs to be visually selected, you'd use `ab%Vc`,
ie. put `%V` before the end of the pattern.

## Fixing the pattern

The below fix shows the correct use of `%V`.
We've put a single `\_s` after it which represents the last character matched.

Because `%V` comes _before_ the `\_s`, it will enforce it's in the visual range.

```
# Old
s/\v\s*$\_s*%V/ /

# Fixed
s/\v\s*$\_s*%V\_s/ /

# Old expanded search term
\v  \s*  $  \_s*  %V

# Fixed expanded search term
\v  \s*  $  \_s*  %V  \_s
```

Run the fixed substitution on the the text below...

```

  Join me
    with the
  the lines
below

 and this one


```

and you'll see now there's _no_ trailing blank lines:

```

  Join me with the the lines below and this one
```

The difference between this and the previous was that we're now matching the newline after "one".

That newline gets converted to a space which causes the blank line following it to get joined into it.

So then we'd be expecting one blank line to remain, but there's zero.

In this and the previous example, there is some other gremlin causing an extra newline to get matched.

## A simpler gremlin example

It's easier to see what's happening with a simpler example:

- put your cursor on the third 'a' on the first line below
- do `v` to enter character wise visual mode
- do `jj` to extend it down two lines (creating a jagged selection)
- do `:s/\v%Va/b/g<enter>`

```
aaaaaaaaaaaa
aaaaaaaaaaaa
aaaaaaaaaaaa
```

You should get:

```
aabbbbbbbbbb
bbbbbbbbbbbb
bbbaaaaaaaaa
```

This makes sense - all the a's that were in the visual selection got changed.

This time we'll replace _blocks_ of a's in the visual selection with `%Va+%Va`,
ie. "one or more a's where the first and last a are in the visual selection".

- put your cursor on the third 'a' on the first line below
- do `v` to enter character wise visual mode
- do `jj` to extend it down two lines (creating a jagged selection)
- do `:s/\v%Va+%Va/b/g<enter>`

```
aaaaaaaaaaaa
aaaaaaaaaaaa
aaaaaaaaaaaa
```

You should get this:

```
aab
b
baaaaaaaaa
```

which again makes sense.

Now we throw a spanner in the works, by changing it from `a` to `\_w`.
Recall that `\_w` means "wordy or newline".

Before we run the example, what would we expect?
If the core of the pattern is `%V  \_w+  %V  \_w` then we'd expect the entire visually selected block (including the newlines)
to get matched in one big block and replaced by a single 'b':

```
aabaaaaaaaaa
^^             the area before the visual selection on the first line
  ^            a single 'b' that replaces the visual selection
   ^^^^^^^^^   the area after the visual selection on the third line
```

Try it and see what happens! Visually select the same jagged area and do: `:s/\v%V\_w+%V\_w/b/<enter>`

```
aaaaaaaaaaaa
aaaaaaaaaaaa
aaaaaaaaaaaa
```

Oh dear! You probably got:

```
aabb[BACKTICK][BACKTICK][BACKTICK]
```

Two weird things here:

- there's _two_ b's
- all the trailing a's from the last line are gone

It turns out that the trailing a's and the newline after them got turned into a 'b' (which explains both observations).
Because the newline was replaced, that caused the backticks to be joined onto the last 'b'.

The issue here is that the searching and replacing seem to be interleaved
(I'm fairly confident of this but haven't found any official verification).

The first replacement reduces the text block significantly, and those trailing a's move into the visual selection.
The regex engine continues processing the remaining a's and matches all the way to the newline at the end of that line.

```
aaaaaaaaaa   aaXXXXXXXX   aabaaaaaaaa  aabXXXXXX...
aaaaaaaaaa   XXXXXXXXXX                XX
aaaaaaaaaa   XXaaaaaaaa

             The a's where             The visual selection
             X's are get               still spans two lines
                                       (represented by X's here)
```

To understand what happened to the visual selection, go back to this example from earlier:

```
A0123456
B0123456
C0123456
D0123456
E0123456
```

Our substitution does reduce the visual selection, but not enough. It still ends up spanning 2 lines
which means that the trailing a's and their newline get included in the visual selection.

## Back to our example

For this example, I don't have a 100% certain explanation for the bug, but I'm fairly confident it's related to this bug.

Recall the issue was that:

```

  Join me
    with the
  the lines
below

 and this one


```

substitutes to:

```

  Join me with the the lines below and this one 
```

You might be thinking that the blank line between "below" and "and this one" is getting deleted
which causes the text below to slide up, causing an extra newline to get matched and replaced.

This isn't the case though. Deleting entire lines usually also updates the visual selection too.

You can test it by deleting the blank line, running the replace and seeing the bug is still there.

If this was the bug you'd expect that adding an equal amount of blank lines above and below "and this one"
would give us the same final result as they cancel out.
But this isn't the case either, we end up with more blank lines below.

What's interesting is that the bug also happens from character visual mode:

- put your cursor on 'J'
- do `v`
- press `j` until being on the "and this one" line
- run the same substitute: `:s/\v\s*$\_s*%V\_s/ /<enter>`
- do `gv` then escape

```

  Join me
    with the
  the lines
below

 and this one


```

You should get the same thing:

```

  Join me with the the lines below and this one 
```

There's no trailing blank lines meaning that 2 of the 3 trailing newline characters got replaced.
The first replace shouldn't have matched any of those trailing newlines because the visual selection
ended on the 'n' of "and".
So it seems very much like it has something to do with `%V` and sliding text.

My theory is that after the first replacement, the visual selection still spans 2 lines,
from the 'J' of "Join" up to 3 characters on the next line as represented by the 'X' in the diagram.

```
  
  Join me with the the lines below and this one
  ^
  XXXXXX...
XXX (up to 3 characters when they exist)
```

Complete lines get removed from the visual selection, but the incomplete lines don't (particularly the last ones).
That leaves two incomplete sections spanning 2 lines (and hence a newline).

After the first replacement, the engine begins matching again and finds the newline after "one".
It runs until the 3 backticks then has to backtrack to match `%V`.
The last character that matches `%V` is the newline represented by the "XXX" above.

Those 2 newlines get replaced with a space leading to a trailing space after "one".

What's not clear though is why it would stop there and not continue destroying the remaining newlines
as they continue to slide up into the visual selection.
Modify the text block to have more trailing newlines and you'll see it still only eats 2 more than expected.
There could be other strange implementation details that are causing the matching to stop at the point.

## Summary of the bug

Using substitute with `%V` can lead to unexpected replacements because search and replace seem to be interleaved.

Because visual selections are static, they don't get updated properly to reflect the operations that happen within them.

This means that a check for whether a character is in the visual selection doesn't reflect whether it was _originally_
in the visual selection.

This is probably a bug and might eventually get fixed.
The example is more to show the issues you'll hit when trying to use complex logic with visual selections
that cause the text in the selection to get changed.

# Summary

Visual mode is a useful tool in some situations, but I'd encourage a mindset of seeing it as a last resort.

There are some cases where they visual mode maes sense but it's much less in vim than other editors and it's good
to train yourself off using it.

For complex logic, visual mode can be dangerous because of the inconsistent unpredictable ways visual areas
get updated when the text inside gets updated.

TODO - go back through other articles and link the content to here
