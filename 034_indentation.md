# Indentation

Vim has built in indent operators `<` and `>` for shifting blocks of text left and right.

# Shiftwidth

The setting which controls the number of spaces in a shift is `shiftwidth`.

You can see what it's set to in your buffer by doing `set shiftwidth?` (it will probably be 2 or 4).

You can set it using the `set` also using the set command, e.g. `set shiftwidth=4`

# Don't forget about good ol' auto-format

In [kata 22](022_formatting.md) we introduced the `=` operator for auto-formatting code.

In a lot of cases if you have this setup correctly, you won't have the need to indent individual lines.

You can also replicate a lot of indentation operations using the `normal` and `global` commands, e.g.

- `:% normal I<space><space>` would right shift the buffer by 2 spaces and:
- `:g/./normal I<space><space>` would right shift the buffer by 2 spaces for _non-empty_ lines

These operators are also a bit inconsistent hence I've introduced them fairly late in the kata.

# Inconsistency

Like `d` and `c`, our indentation operators `<` and `>` can be combined with text objects.

For example `>ip` right shifts the current paragraph.

Likewise `<<` and `>>` use the current line as the default text object
(see [kata 6 - operation shortcuts](006_operation_shortcuts)).

However once _counts_ are introduced, things don't work the way you'd expect.

You would probably expect:

- `2>>`: right shift the current line twice
- `3>ip`: right shift the current paragraph thrice

But sadly sadly this is not the case :sad-parrot:

In both examples the number is not intepreted as a count,
instead it's the number of lines starting from the cursor to indent.

So what they actually do:

- `2>>`: right shift this line and the below line once
- `3>ip`: right shift this paragraph and the 3 lines below it once

The paragraph based one is something you'd almost never want to do.

I haven't seen a good justification for the choice to make indent work this way
and I feel it's a flaw in vim's design,
particularly when there are already simple ways to achieve indenting a block of lines,
e.g. `>3j` to indent this and the 3 lines under.

# Multiple shifts

So then the question is:

> How do I shift multiple times?

## Shift once and use `.`

You can shift it once (e.g. `>ip`) and then hammer the dot operator until it's where you want.

This isn't that bad. For large indentations (like 3+ jumps) it's too hard to visually determine
how many jumps you want to do and if you get it wrong, doing undo will undo the whole things.

At least with hammering dot, it's easy to course correct if you go to far.

## Visually select

Visually selected text treats counts as actual counts.

For example if you visually highlight a line with `V` and do `5>` it will indent the current line 5 times.

So oddly they decided to make shift operators in visual mode consistent with the other operators like `d`,
but inconsistent with itself from normal mode.

## Use an ex command

There are actually ex commands for indenting. The general form is:

```
# Shift the range left or right by the number of carets
:[RANGE] [1 OR MORE >'s or <'s]

# Shift the relative range by the number of carets
:[1 OR MORE >'s or <'s] [NUMBER OF LINES]
```

Some examples for the first form:

- `:% >>` - right shift the entire buffer twice
- `:-1,+1 <<<` - left shift the line above, this line, and the line below below, three times

Some examples of the second form:

- `:>>> 5` - right shift a 5 line block starting from the current line three times
- `:<< 2` - left shift this and the line below twice

Unfortunately you can't use negative numbers with the second form to represent a block of
lines going up.

Also these ex commands are NOT dot repeatable :sad-parrot:

# Exercises

To make sure we've all got the same indentation, do `set shiftwidth=2`.

## Exercise 1

Make the second and fourth blocks below line up with the rest:

- put your cursor on the second paragraph ("The story of the wild Boban")
- do `>ip` (right shift inside paragraph)
- hammer `.` until it visually lines up
- (looks like we needed 4 shifts overall)
- put your cursor on the last paragraph ("In a secluded...")
- do `vip` to highlight the paragaph
- do `4>` to shift it 4 times

```scala
def paragraph: String = {
  """
        |Gather around children,
        |to hear a story.

|The story of the wild Boban.
|A story as old as time...

        |It all started in the mountains
        |of Albania.

|In a secluded monastery high up in the mountains, 
|a young boy was being raised...

  """.stripMargin
}
```

## Exercise 2

We'll do the same exercise but using the first form of the ex commands:

```
:[RANGE] [<'s or >'s]
```

- put your cursor on the first line of the second block
- do `.,+1 >>>><enter>`
- (unfortunately we can't use `.` to repeat this)
- put your cursor on the second line of the second block 
- do `.,-1 >>>><enter>`
- do `u` to undo
- do `vip:` to visually highlight the block then convert it to a range
- add `>>>><enter>` onto the command

```scala
def paragraph: String = {
  """
        |Gather around children,
        |to hear a story.

|The story of the wild Boban.
|A story as old as time...

        |It all started in the mountains
        |of Albania.

|In a secluded monastery high up in the mountains, 
|a young boy was being raised...

  """.stripMargin
}
```

## Exercise 3

We'll do the same exercise using the second form of the ex command:

```
:[<'s or >'s] [NUMBER OF LINES]
```

- put your cursor on the first line of the second block
- do `:>> 4`
- put your cursor on the first line of the fourth block
- do `:>> 4`

```scala
def paragraph: String = {
  """
        |Gather around children,
        |to hear a story.

    |The story of the wild Boban.
    |A story as old as time...

        |It all started in the mountains
        |of Albania.

|In a secluded monastery high up in the mountains, 
|a young boy was being raised...

  """.stripMargin
}
```

# Conclusion

Overall it's a bit of a mess.

We have many ways to shift code:

- text objects (e.g. `>ip...` or `vip4>`)
- ranges (e.g. `:.,+1 >>>>`) 
- line counts (e.g. `2>>...` or `:>> 4`)

It's a shame that they are all so different from each other and none adhere to the more general
"count + operator + text object" form we're used to (except visual mode).

Overall I'd recommend just learning the text object approach as you'll be able to apply that to any line based
text objects that get added through plugins (e.g. `e` for the buffer, `l` for the current line) and you can use
that form to replicate the other forms with about the same number of key presses, e.g. `:.,+3 >>>` is `>3j..`.

It also makes your changes more reusable on other blocks with `.` which you don't get with ex commands.

Further in [kata 37](037_indentation_text_objects.md) we'll actually look at indentation based text objects.
Often you're wanting to indent a bunch of lines that are all at the same level and to use those ex commands
you've going to be manually counting the lines.
Indentation based text objects make that a lot easier, you just say: "right shift everything at this level of indentation".
So that is another benefit of going with the text object based approach.

Finally don't forget that there is also the `=` operator for formatting text objects.
Chances are you're indenting code to bring it into adherence with some basic formatting rules,
so formatting some broader text object might get you what you want without having to fiddly with smaller blocks of code.
