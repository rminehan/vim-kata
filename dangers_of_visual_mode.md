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

## Auto-populating from visual character mode

Notice that when vim populates the range for you, it does this by using the line number of the start
and end anchors for your most recent selection.

```
'<,'>
```

It doesn't actually matter whether you were in character, line or block mode.
All that matters is the lines for the anchors.

So the ex command you run doesn't really care about what you have selected,
it cares about the line range from what you have selected. To make this point:

- put your cursor on the third 'a'
- do `v`
- press `jjj` to extend it downwards
- do `:normal gU$<enter>` (to upper case from the cursor to the end of the line)

```
aaaaaa
bbbbbb
cccccc
dddddd
```

Notice how the entirety of each line got uppercased (even the first and last).
This is because `'<` and `'>` got converted to line numbers,
then that range of lines had `normal gU$` applied to them.

To hammer it home, you can repeat the above with visual block mode (`<c-v>`).

## Prison breaks

_Another_ confusing aspect of this is that the substitute range defines the allowed _starting_
positions for matches, but there's nothing that stops a match "breaking out".

For example:

- put your cursor somewhere on the first line
- do `V:` start an ex command on that line
- do `s/\v\_w+/X/<enter>`  (here `\_w` means "wordy or newline")

```
aaaaaaa
aaaaaaa
#
```

Only the first line of 'a's was in the range defined by `'<,'>`.

The very first 'a' on that lines started a match.
The `\w+` then ran to the last 'a' on that line, then over the newline,
then onwards onto the next line at which points it's done its "prison break".

It consumed all the 'a' on the next line and the newline after.
It's finally halted by the '#'.

So the final text should be:

```
X#
```

Most of the time you don't need to worry about this because you're _not_ using atoms that match newlines.
That means all matches must end on the same line they started on.
Hence if a match starts within the line range, it must end in the line range.

Once you introduce atoms like `\_*` that match newlines, things get more confusing.
Patterns can prison break out of the substitution range, and if you want to limit them you need
to use `%` based matches, in particular `%V` which the next section discusses in detail.

See also [this related article](why_avoid_newline.md) for info about matching newlines.

# Limiting regex searches

[Kata 43](043_advanced_regex_1_restricting_search_space.md) introduces many mechanisms to restrict matches
in a search based on context within a buffer.

One such mechanism is `%V` (or `\%V` in magic mode) which only matches if the engine is inside the most
recently selected visual block (see `:help /\%V`).

It turns out there's some "bugs" or (at least unexpected behaviors) using this with the `substitute` command.

First though we need to clarify how `%V` works with visually selected lines.

## Substituting and %V

As explained above, when you visually select something and hit `:`, vim populates the ex command with a line range,
based on the positions of the start and end anchors.
It doesn't matter what kind of visual selection it was, only the anchor positions affect those line numbers.

For the `substitute` command, that range defines which characters to consider as starting points when
searching for matches to replace.
The range is line based, meaning that `substitute` is always scanning on _blocks of lines_.

The line range isn't affected by the kind of visual selection you had, but `%V` definitely _is_ affected.

For example:

- put your cursor on the third 'a' on line 2
- do `v`
- do `j`
- do `:s/\v%Va/X/g`

```
aaaaaa
aaaaaa
aaaaaa
aaaaaa
```

You should get:

```
aaaaaa
aaXXXX
XXXaaa
aaaaaa
```

The range specified for the engine to consider overall was the two middle lines
and `%V` limited those characters to the ones that were selected.

```
aaaaaa\naaaaaa\naaaaaa\naaaaaa\n
        ----------------            Range defined by '<,'>
          ---------                 Visual selection
```

Repeat the same steps on this block but use `V` or `<c-v>` instead of `v`.
You'll see it makes a difference to what got replaced.

```
aaaaaa
aaaaaa
aaaaaa
aaaaaa
```

For another more interesting example:

- create the same character visual selection as before
- press escape
- put your cursor on the the top line of a's
- do `:.,+1 s/\v%Va/X/g<enter>`

```
aaaaaa
aaaaaa
aaaaaa
aaaaaa
```

You should get:

```
aaaaaa
aaXXXX
aaaaaa
aaaaaa
```

The difference here was that we didn't let the visual selection drive the line range for the substitute command.
We did `.,+1` which means "this line and the one below".
We did it in a way that didn't override the previous visual selection.
`gv` should still show that same jagged selection you originally made.

```
aaaaaa\naaaaaa\naaaaaa\naaaaaa\n
----------------                    Range defined by .,+1
          ---------                 Visual selection
          ------                    Overlap - any a's in here become X's
```

When we look at a confusing bugs later, this distinction between the line range and the visually selected area
will become relevant.

# Deep dive - hunting a most peculiar bug

## Pre-amble

This section is a deep dive into a fairly peculiar bug related to vim's regex and visual mode.

It's unlikely this bug will ever effect you as:

- it requires a fairly artificial setup
- the consequences of it aren't particularly bad and often you wouldn't even realize it happened
- it might get fixed eventually

So the point of reading this section is not to make you aware of the bug,
it's to deepen your knowledge and make you more aware of the kinds of edge cases you'd hit
when you do complex things with visual mode.

If you don't want to do the deep dive then you're finished with this article -  just jump to the final conclusion!

## Kata 49 - exercises 1 and 2

[Kata 49](049_advanced_regex_7_multiline.md) exercises 1 and 2 show the effect of including or not including
`%V` at the back of a pattern used in a substitute.
Without it, we get a prison break situation.

The original aim of exercise 1 was to cleanup trailing whitespace and blank lines within a visually selected block.

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
That's shouldn't be the case though.

Let's track what the engine should do based on our understanding:

```
\n  Join me\n    with the  \n  the lines\nbelow\n\n and this one\n\n\n```
    -----------------------------------------------------------------------
    ^             Visually selected                             $ matches
    ^                                                           -->--> \_s*
    ^                                                           --<--< backtrack for %V
    ^                                                           %V matches the \n but doesn't consume it
    ^                                                          ^
    ------------------------------------------------------------ consumes up to 'e'
                          Expected final match
```

- at the newline after "one", `\s*` will have a zero width match
- `$` will match before the newline (zero width)
- `\_s*` will match the newline and run all the way to the 3 backticks
- the `%V` won't match at the first backtick as it's not in the original visual selection
- the engine will backtrack until the character _following_ its position is in the visual selection
- it stops _on_ the newline after "one" and the `%V` has a zero width match there (not consuming the newline)

The engine should finish just before the newline after "one" and _should not have_ replaced it.
The last character consumed should be the 'e' from "one".

So the newline after "one" shouldn't have gotten matched.
That means there _should_ still be 3 newlines which translates visually to two blank lines.

It turns out the newline does get matched but for a different reason,
but before getting to that, we should fix the subtle issue with how we're using `%V`.

## The character _before_ the end

If you do `:help /\%V` it says;

```
\%V     Match inside the Visual area.
        When Visual mode has already been stopped match in the area that gv would reselect.
        This is a zero-width match.
        To make sure the whole pattern is inside the Visual area put it at the start and just before the end of
        the pattern
```

Note the part about _just before the end of the pattern_.

What it means is that `%V` shouldn't be right at the end of your pattern,
but it should be matching the last character consumed in your pattern.

It's easier to see this subtle distinction with an example.
Below we want to replace "abc" with "ABC" when it's completely visually selected:

- put your cursor on the first 'a'
- do `vjll` to extend the selection across both "abc"s
- do `:s/\v%Vabc%V/ABC/g<enter>`

```
abcde
abcde
```

Note how it replaced the top one but not the bottom.

The reason is because the second `%V` in the pattern is checking the character _after_ 'c'.

On the second line, `%Vabc` consumed "abc" and now the engine is on 'd'.
It runs `%V` and _doesn't_ get a match because 'd' wasn't visually selected.

The bug is that we didn't put `%V` "just before the end of the pattern" to use the language of the help page.

This looks weird but it makes sense. After all to make sure the 'a' is in the visual selection, we put `%V` _before it_.
It wouldn't make sense for that `%V` to look forward, but the one for 'c' to look backward.

## Fixing the pattern

Applying this to our example, the below fix shows the correct use of `%V`.
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

Previously it was:

```

  Join me with the the lines below and this one

```

So the change has caused an extra newline to get matched.

```
  Join me\n    with the\n  the lines\nbelow\n\n and this one\n\n\n
                                                            --        matched before we added \_s
                                                            ----      matched once we added \_s

```

In both cases it's wrong and should be matching one less newline.

There is some other gremlin causing an extra newline to get matched...

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

## What's going on?

Two weird things here:

- there's _two_ b's
- all the trailing a's from the last line are gone

It turns out that the trailing a's and the newline after them got turned into a 'b' (which explains both observations).
Because the newline was replaced, that caused the backticks to be joined onto the last 'b'.

The issue here is that the searching and replacing seem to be interleaved
(I'm fairly confident of this but haven't found any official verification).

It finds a match, replaces it, finds another match, replaces it, etc...

```
aaaaaaaaaa   aaXXXXXXXX   aabaaaaaaaa  aabXXXXXX...
aaaaaaaaaa   XXXXXXXXXX                XX
aaaaaaaaaa   XXaaaaaaaa

             The a's where             The visual selection
             X's are get               still spans two lines
              matched                 (represented by X's here)
```

The first replacement reduces the text block significantly, and those trailing a's slide up into the visual selection
(even though they originally _weren't_ in the selection).
The regex engine continues processing the remaining a's and matches all the way to the newline at the end of that line.

To understand what happened to the visual selection, have another read through the example from earlier that used:

```
A0123456
B0123456
C0123456
D0123456
E0123456
```

It should show you that our substitution does reduce the visual selection, but not enough.
It still ends up spanning 2 lines which means that the trailing a's and their newline get included in the visual selection.

## Back to our example

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

### False theory

You might be thinking that the blank line between "below" and "and this one" is getting deleted
which causes the text below it to slide up, causing an extra newline to get matched and replaced.

This isn't the case though. Deleting entire lines usually also updates the visual selection too.

You can test it by deleting the blank line, running the replace and seeing the bug is still there.

Likewise if this was the bug you'd expect that adding an equal amount of blank lines above and below "and this one"
would give us the same final result as they cancel out.
But this isn't the case either, we end up with more blank lines below - adding more above doesn't seem to amplify
the bug to consume more space below.

### Character visual mode

What's interesting is that the bug also happens from character visual mode when we explicitly exclude the newline
after "one":

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

### Remaining jagged lines

What we observed in the early example:

```
A0123456
B0123456
C0123456
D0123456
E0123456
```

was that the jagged start and end chunks of visual selections don't seem to get modified as text is removed.
Only complete lines seem to be removed.

Let's reduce our example to just the first couple of lines:

- put your cursor on 'J'
- do `vj`
- do `:s/\v%V\s*$\_s*%V\_s/ /<enter>`
- do `gv`

```
  Join me
    with the


```

you should get:

```
  Join me  with the 
```

and `gv` should be a jagged selection starting at 'J' and ending on the last tilda.

Repeat it on this block but pressing `j` twice when extending the selection:

```
  Join me
    with the
  the lines


```

Again you should get two jagged lines in the selection.
In this case, the internal line was deleted out of the visual selection.

The point of this is to make you see that as the lines get gradually joined together,
the visual selection shrinks removing entire lines,
_but_ when it gets to what we expect to be the very last step where all the lines are joined,
the visual select still spans two lines represented by the 'X's below:

```
  
  Join me with the the lines below and this one
  ^
  XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
XXX (up to 3 characters when they exist)
```

The engine begins matching again and finds the newline after "one".
It runs until the 3 backticks then has to backtrack to match `%V`.

```
 Join me with the the lines below and this one\n\n\n[BACKTICK]...
 -------------------------------------------------                 visual selection
                                              -->-->               \_s* runs until a backtick (consumes first newline)
                                                <--<               backtrack until %V matches (before the middle newline)
                                                --                 \_s (consumes middle newline)

                                              ----      final match is the two newline characters
```

The 2D visualization represents that the visual selection would take up to 3 characters on the next line
when they're there. It's a linewise concept.
The effect is that the middle newline is now considered "selected" when it _shouldn't_ be.
This is the root of the bug - the middle newline slid up into the visual selection.

Hence the engine backtracks until the middle newline which matches `%V`, then `\_s` consumes the middle newline.

The first and second newlines get replaced with a space, leaving just the third newline between the text and the backticks.

### Why doesn't it keep going?

If this theory is correct, then wouldn't the engine continue to munch up newlines?

After the first and second newline got matched and replaced by space, shouldn't the third newline
and the 3 tildas slide up into the visual selection?
If they're in the visual selection, then they should get matched to `%V\_s` and hence get replaced with a space.

```
 Join me with the the lines below and this one \n[BACKTICK x 3]
 --------------------------------------------------------------   visual selection
                                               ^^ match???
                                               Replace with space???

                                           one  [BACKTICK x 3]   ???
```

Yet we don't see "one" then two spaces, then 3 backticks.

To emphasize the point, below is an example with 5 trailing newlines (which visually is 4 blank lines):

- put your cursor on 'J'
- do `v`
- extend the selection down to "and this one"
- do `:s/\v%V\s*$\_s*%V\_s/ /<enter>` as usual

```

  Join me
    with the
  the lines
below

 and this one




```

It should have just 3 trailing newlines (which visually is 2 blank lines) and a single space. Only 2 got replaced.

```
  Join me with the the lines below and this one\n\n\n\n\n[BACKTICK x 3]
                                               ----  just 2 get replaced
```

### Remember ranges?

So why isn't the bug affecting these other trailing newlines? Is it just getting tired and going home?

The answer comes from the range generated by `'<,'>` when you hit `:`.

Recall from a previous section how the range passed to a substitute command determines the overall text which the engine
will consider when _starting_ matches.

Matches can "prison break" out of that and end outside that range, but they can only _start_ in that range.

Our visual selection started on the 'J' and stopped on the 'n' which translates to lines:

```

  Join me              |
    with the           |
  the lines            |  line range
below                  |  '<, '>
                       |
 and this one          |




```

In 1D:

```
\n  Join me\n    with the\n  the lines\nbelow\n\n and this one\n\n\n\n\n
    ------------------------------------------------------------
              '<,'> defines these allowed starting points
```

The newline just after "one" _is_ a valid starting point.
A match will start on that newline, and he and his brother will prison break outside the initial starting range
because by the time the engine is processing them, his brother has slid into the visual selection
and will match that final `%\_s`.

That match completes and then the regex engine will stop because it's exhausted all the characters
within the allowed starting positions.
That means the other newlines can't get matched.

It's a very cunning prison break that allows just one extra newline to sneak through.
Most wardens wouldn't be able to spot that loophole!

### What about linewise visual mode?

The analysis above was for the jagged character visual mode.

It relied on issues with the jagged start and end sections of visual selections.

Linewise visual selections shouldn't suffer from that though right?

- put your cursor somewhere on "000"
- do `Vj<escape>` to select two lines
- do `k` to go up to the "000" line
- do `dd` to kill it
- do `gv` - what do you see? (should just be "111" selected)

```
000
111
222
```

Let's try a variation where we delete the bottom line:

- put your cursor somewhere on "000"
- do `Vj<escape>` to select two lines
- do `dd` to kill the "111" line
- do `gv` - what do you see? (should now be _two_ rows)

```
000
111
222
```

:not-very-consistent-parrot:

In particular notice that when the _bottom_ line gets deleted, the visual selection doesn't shrink.
That's applicable to our bug.

Eventually the visual selection will shrink to the point where the final expected flattening step will be performed.
Will the visual selection shrink? We can emulate something similar by looking at our example near the last step:

- put your cursor on the first line
- do `Vjj`
- do `:s/\v%V\s*$\_s*%V\_s/ /`
- do `gv`

```

  Join me with the the lines below

 and this one


```

Again you can see it killed too many newlines _and_ the visual selection is the current and next line.

My theory is that when the bottom line gets removed, the visual selection doesn't shrink as expected,
catching that extra newline.

## Other bugs?

The interleaving approach of `substitute` would create potential for other bugs.

Let's have a quick poke around:

## Lookbehind

When a new match starts during a `substitute`, do lookbehinds see the old or new value?

Suppose we did something like:

- replace 'a' with 'X'
- replace 'b' with 'X' if there's an 'X' before it

In text like this, would the 'b' get replaced?

```
ab
```

If the first 'a' is replaced by an 'X' before the 'b' is processed,
then maybe the 'b' will be considered to have an 'X' before it.

- put your cursor on the line
- do `V:s/\v(a|X@<=b)/X/g<enter>`

That pattern means: 'a' OR 'b' with an 'X' before it (positive lookbehind).

It turns out only the 'a' gets replaced (which is a relief).

I think this means that the engine is using the original chunk of text for matching logic,
but for context related logic (like `%V`) it's projecting what the final text would look like
and applying changes to the visual area.

### Single line visual selections

Can we create a black hole singularity of finite width that relentlessly sucks in the characters after it?

Let's delete any digit characters in our visual selection:

- put your cursor on the '3'
- do `vll` to select "345"
- do `:s/\v%V\d//g`

```
0123456789
```

Maybe you expecting "6789" to also slide to the left into the death zone but they don't.

You might think it's because the visual selection shrinks by one character as '3', '4' and '5'
get killed off, but that's not the case.
If you run `gv` after the substitution you'll see it's on "456".

If you change the text to have multiple lines and make your visual selection go from '3' to '3'
and use `\_d` instead you'll get the bug.
It's the same kind of example as our previous one with the "aaaaaa" lines.

```
0123456789
0123456789
```

A bug for the first simple case would get spotted more quickly when testing `%V` so I'm not surprised it works.
What's more confusing is why it works fine but the multiline one doesn't work well.
There's a bug in our bug...
Perhaps when the engine sees `\_*` patterns, it changes mode internally and a different code path gets used.
That different code path could be responsible for the bug.

## Summary of the bug(s)

Our first bug was the incorrect use of `%V`. It was causing one _too few_ newlines to get matched.
The second bug was causing one _too many_ newlines to get matched.
Hence the two of them cancelled out.

Using substitute with `%V` can lead to unexpected replacements because search and replace seem to be interleaved.

Instead of using the original visual selection and character positions prior to any substitution,
it instead does a more stateful iterative process, replacing characters and updating the visual selection along the way.

As we've seen, this process of updating the visual selection is very inaccurate and inconsistent
leaving ample room for bugs.
In our case it would cause an extra newline to get sucked into the visual selection and get matched.

The other newlines got sucked into the visual area, but because they weren't in the original substitution line range,
new matches didn't start on them.

When we tried to replicate the bug in simpler scenarios it thankfully didn't manifest.

Overall this is probably a bug with neovim and might eventually get fixed.
I've posted a bug report [here](https://github.com/neovim/neovim/issues/13838)
and maybe the neovim developers can shed more light on it.
It was a pretty benign bug that was just chopping off one too many newlines.
Hopefully though you learnt more about the nuances of regex engines and can see that combining complex stateful
logic with visual mode is likely to hit bugs.

# Summary

Visual mode is a useful tool in some situations, but I'd encourage a mindset of seeing it as a last resort.

There are some cases where visual mode makes the most sense,
but it's much less in vim than other editors and it's good to train yourself off using it.

For complex logic, visual mode can be dangerous because of the inconsistent unpredictable ways visual areas
get updated when the text inside gets updated.
