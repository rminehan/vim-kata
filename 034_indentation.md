# Indentation

Vim has built in indent operators `<` and `>` for shifting blocks of text left and right.

It turns out there's many ways to indent text which can be more confusing than helpful.

Overall I feel the best approach is the classic "operator + text object" style combined with `.`.

# Shiftwidth

The setting which controls the number of spaces in a shift is `shiftwidth`.

You can see what it's set to in your buffer by doing `set shiftwidth?` (it will probably be 2 or 4).

You can set it using the `set` command, e.g. `set shiftwidth=4`

# Exercises

To make sure we've all got the same indentation, do `set shiftwidth=2`.

## Exercise 1

For this exercise we'll use "operator + text object" style.

Make the second and fourth blocks below line up with the rest:

- put your cursor on the second paragraph ("The story of the wild Boban")
- do `>ip` (right shift inside paragraph)
- hammer `.` until it visually lines up
- (looks like we needed 4 shifts overall)
- put your cursor on the last paragraph ("In a secluded...")
- do `>j` (right shift this and the line below)
- hammer `.` until it visually lines up

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

## Exercise 2 - counts?

In exercise 1 we had to indent blocks 4 times.

Let's try introducing a count to achieve that in one command rather than having to hammer on `.`.

- put your cursor on the second paragraph ("The story of the wild Boban")
- do `4>ip`
- (that didn't do what we wanted at all)
- hit `u` to undo
- do `2>>` then hammer on `.`
- (that should do the right thing)

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

So what happened here?

It turns out that "counts" don't work as expected with indent when used in "count + operator + text object" mode.
We were expecting it to indent 4 times, but instead it affected the _number of lines_ being indented.

In the second example we did `2>>` which means "right indent 2 lines once" and not "indent the current line twice"
as you may have expected.
So the "count" actually refers to the number of lines.

The first example `4>ip` doesn't have a clear meaning.
It is something like "right indent the paragraph and then 4 more lines".
If you try it with other counts (like 5) though you'll see it doesn't quite work like this.

Overall "count + `>` + text object" is confusing and not useful.
The form "count + `>>`" can sometimes be useful and it's at least easier to reason about what it does.

A note on terminology:
In vim's help docs and other resources, the number before the "operator + text object" form is still called a "count"
even if it doesn't have a count intuition.
"count" is really referring to the syntactic position and it's up to plugin writers whether they want to use it in
a way that intuitively fits a count concept.

## Exercise 3 - visual mode

Recall that visual mode relates to "operator + text object" in that the visual selection defines an ad-hoc text object
so when we type an operator, it's automatically applied to the visual selection.

Oddly when indenting in visual mode, counts are treated as repetitions rather than a number of lines.

- visually select the block we want to indent with `vip`
- do `>`
- do `gv` to reselect the block
- do `3>`

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

Usually I try to avoid visual mode but this is one case where you can get a more idiomatic vim experience
consistent with the other operators.

Other operators will also repurpose vim's count operator to have a different meaning.

# Other ways?

It turns out there's other ways to achieve indentation:

- ex command form 1: `[RANGE] [1 OR MORE >'s OR <'s]`
- ex command form 2: `[1 OR MORE >'s OR <'s] [NUMBER OF LINES]`
- insert mode shortcuts
  - left indent `<c-d>`
  - right indent `<c-t>`
- normal/global commands, e.g.
  - `:%normal I<space><space>` indents the whole buffer two spaces
  - `:g/./normal I<space><space>` would right shift the buffer by 2 spaces for _non-empty_ lines

For this kata I won't bother covering them because I think it's better to just learn one way well
and overall I feel the "operator + text object" form is the most powerful and easiest to remember given
how much we've already covered it and will continue to do so!

In particular in [kata 37](037_indentation_text_objects.md) we'll actually look at text objects specifically
built to represent indentation levels.
These work beautifully with the indent operators but you wouldn't be able to leverage them with the methods above.

# Don't forget about good ol' auto-format

In [kata 22](022_formatting.md) we introduced the `=` operator for auto-formatting code.

Often you're indenting source code to bring it into adherence with some basic formatting rules,
so auto-formatting some broader text object (like a paragraph or buffer) might get you what you want without
having to fiddle with smaller blocks of code.

# Conclusion

Overall it's a bit of a mess.
There's many ways to perform indentation and there are inconsistencies in how they're implemented.

It's a shame that they are all so different from each other and none adhere to the more general
"count + operator + text object" form in an intuitive way (except visual mode).
It's because of this that I've put the indent operators later in the kata's,
as I didn't want to confuse the operator + text object pattern.

Overall I'd recommend just learning the text object approach as you'll be able to apply that to any line based
text objects that get added through plugins (e.g. `e` for the buffer, `l` for the current line).

When you want repetition just hammer on the dot operator.
That's not too much slower than using a count and it allows you to course correct more easily if you got the count wrong.

Finally don't forget that there is also the `=` operator for formatting text objects.
