# Indentation text objects

This kata introduces text objects to group up lines based on relative indentation.

We often find ourselves in situations like:

> Comment out all lines at the same indentation level as my current line or deeper

> Left indent all lines at the same indentation level as my current line or deeper

This plugin introduces a text object that makes these kinds of changes dead simple.

# Plugins

As with the previous two kata, we require installing plugins to get these additional text objects.

In our case we need [vim-indent-obj](https://github.com/michaeljsmith/vim-indent-object).

# The indentation text objects

The plugin is based around a text object representing lines at the same or deeper indentation as the current line.

All the permutations of it are just related to including leading and trailing parent lines.

From the perspective of including or excluding:

- `i*` - exclude it ("inside")
- `a*` - include it ("around")

From the perspective of leading vs trailing:

- `*i` - just leading parent line (and trailing whitespace)
- `*I` - leading and trailing parent line

Let's look at some examples:

```
Level 1
  Level 2a
  Level 2b       ----------------|ai --|aI
                                 |     |
    Level 3a     --------|ii     |     |
                         |iI     |     |
    Level 3b (cursor)    |       |     |
      Level 4a           |       |     |
      Level 4b           |       |     |
    Level 3c             |       |     |
                         |       |     |
    Level 3d    ---------|       |     |
                -----------------|     |
  Level 2c      -----------------------|
    Level 3e
```

Note that the `|` guides to the right of the diagram actually interfere with the indentation logic of the diagram.
If you want to replicate the results above use the version below without guides.

Above the cursor is anywhere on "Level 3b".

- `ii` represents 3a to 3d. 3e is excluded because 2c splits 3d from 3e
- `iI` is always the same as `ii` because they're both excluding all above/below lines
- `ai` represents 2b and all lines underneath up to but not including 2c.
       That means it captures the trailing whitespace after 3d.
       Note that 2a doesn't get included - just one parent line.
- `aI` represents 2b to 2c as they bracket that indentation

Use the below to double check the indentation levels for yourselves
(as I know most of you are millenials who distrust authority).
For example do `vii` to visually see what it looks like.
Also notice how it doesn't really matter which level 3 line it's on, you'll get the same thing.

```
Level 1
  Level 2a
  Level 2b

    Level 3a

    Level 3b
      Level 4a
      Level 4b
    Level 3c

    Level 3d

  Level 2c
    Level 3e
```

# Discontiguous blocks

A nice thing about these text objects is that they don't get broken apart by blank lines.

For coding, there are often blank lines inserted between code at the same level of abstraction, e.g.

```scala
if (enxhellsABoban) {
  // Instruction 1
  ...

  // Instruction 2
  ...
}
else {
  throw new Exception("Internal logic error")
}
```

Instructions 1 and 2 are logically at the same level of abstraction but separated by a blank line.

Paragraph based text objects would be useless here, but

- `ii` or `iI` would capture the 2 instructions and all comments
- `aI` would capture the instructions, comments and the `if` and closing `}`

So if your codebase tends to represent logical structure using indentation levels,
then the indentation text objects start to become a bit "language aware".

# Back to `<` and `>`

In the conclusion of [kata 34](034_indentation.md) I mentioned how indentation operators work really nicely
with the indentation text objects and that's a benefit of sticking with the "operator + text object" style
rather than learning the 37 other ways to shift text in vim.

Exercise 1 below demonstrates this point.

# Exercises

We'll be doing some shifting operations below with `<` and `>`.

To get everyone on the same page, do:

```
set shiftwidth=2
```

## Exercise 1

Make the level numbers line up with their indentation level below.

There's two misaligned blocks indicated by the `|` guides.

- put your cursor anywhere on a level 3 line in the first block (3b, 3c or 3d)
- do `<ii`
- put your cursor anywhere on a level 2 line in the second block (2c or 2d)
- do `<ii` or `.`
- (whoops, we wanted to include the parent line - do `u` to undo and reposition your cursor)
- do `<ai`
- (whoops, that got the leading parent, but we forgot the trailing parent 1c - do `u` to undo and reposition your cursor)
- do `<aI`

```
Level 1a
  Level 2a
    Level 3a
      Level 3b       |
        Level 4a     |
        Level 4b     |
      Level 3c       |
      Level 3d       |
  Level 2b
  Level 1b     |
    Level 2c   |
    Level 2d   |
  Level 1c     |
```

## Exercise 2

The if statements in the code below got paired with the wrong else statements.

Switch them back!

- put your cursor on one of the lines inside the first if statement (e.g. "// _Probably_ not a Boban")
- do `"tdaI` which is:
  - `"t` - into register `t` (for "technical")
  - `d` - delete or cut in this case
  - `aI` - everything at this level of indentation or deeper plus the lines that bracket this in (the `if` and `}`)
- put your cursor anywhere inside the next if statement (e.g. "// :hmmm-parrot: ...")
- do `"wdai` which is:
  - `"w` - into register `w` (for "wisdom")
  - `d` - delete or cut in this case
  - `ai` - everything at this level or deeper and just the header line as the closing `}` is already captured
- do `"tP` to paste from register `t` on the cursor
- move your cursor back up to the line of the first `else`
- do `"wP` to paste from register `w` on the cursor

```scala
def isBoban(person: Person): Boolean = {
  val technicalBobanity = if (person.usesScala) {
    // _Probably_ not a Boban...

    if (person.triesToSneakInPython) {
      true
    }
    ...

  }
  else {
    // Not arguing with Rohan is a good sign
    ...
  }

  val wisdomBobanity = if (person.arguesWith(rohan)) {
    // :hmmm-parrot: Check for parrot compliance
    if (person.arguesAbout(partyParrots)) { ... }

    ... }
  else {
    // Hmmm... not a scala user eh...
    // Check for Haskell and Clojure...
    ...
  }
}
```

# Exercise 3 - level 1 indentation

The text object behave a little differently when you're right against the left margin.

Concepts like `ai` and `aI` don't make sense here because there is no parent.

Also `ii` would represent the entire buffer as _everything_ is at the same level of indentation
or deeper when you're not indented.

So the text objects change meaning slightly. We'll understand this by testing out:

- put your cursor on 1b
- do `vii` (the paragraph should highlight)
- hit escape and reposition your cursor
- do `vai` (the paragraph and one line of whitespace above should highlight)
- hit escape and reposition your cursor
- do `vaI` (the paragraph and one line of whitespace above and below should highlight)

```


Level 1a
  Level 2a
  Level 2b
Level 1b
  Level 2c

Level 1c
  Level 2d
    Level 3a


```

From the [help page](https://github.com/michaeljsmith/vim-indent-object/blob/master/doc/indent-object.txt):

> When scanning code blocks, the plugin usually ignores blank lines.
> There is an exception to this, however, when the block being selected is not indented.
> In this case if blank lines are ignored, then the entire file would be selected.
> Instead when code at the top level is being indented blank lines are considered to delimit the block.

So this is a deliberate features.

The effect is that the:

- `ii` is equivalent to `ip` (inside paragraph)
- `ai` is like `ip` with an extra line of whitespace above
- `aI` is like `ap` with an extra line of whitespace above

The help also says we can turn this off with:

```vim
let g:indent_object_except_first_level = 0
```

Run the above ex command and repeat the exercise
You should select the entire buffer each time.
Also you can run the commands from anywhere in the buffer, it won't make a difference.

# Conclusion

Source code usually puts concepts at the same level of abstraction at the same indentation level.

This makes these indentation based text objects very useful for capturing that text without having to
worry about being broken by blank lines the way the paragraph object is.

This particular text object also allows grabbing the leading parent with `ai` and leading + trailing parent with `aI`.

# Further reading

You might also find [vim-indentwise](https://github.com/jeetsukumaran/vim-indentwise) useful.
It defines a lot of motions based on indentation levels, e.g. `[-` and `[+` for moving to the next line
of lesser or greater indentation respectively.
