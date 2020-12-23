# Global Blocks

There is another form for the normal command which we haven't covered,
and that's for "blocks".

```
[RANGE] global /[START PATTERN]/[OFFSET], /[END PATTERN]/[OFFSET] [EX COMMAND]
```

(We'll see below that this is a lie but roll with it for now)

You can pass _two_ patterns to the normal command to define "blocks" in your code:

- the start pattern + offset describes the start of each block
- the end pattern + offset describes the end of each block

The ex command gets applied to the lines within these blocks. 

# Example 

Let's uppercase the code between the start and end tags below.

The start/end markers are `<start>`/`<end>`.
The carets are magic characters (word boundaries) so we'll use `\V` (very no magic) to remove all the
magic and have them as literal carets.
Also we'll put start line anchors (`^`) on them so that they don't match the `<start>` and `<end>` words
in this paragraph!
That makes the patterns `^\V<start>` and `^\V<end>`.
Note that we've put the vertical carets _before_ the `\V` so that they keep their magic.

We don't want to uppercase the tags themselves, so:

- the start tag has an offset of `+1` (start the block one line below the match)
- the end tag has an offset of `-1` (end the block one line above the match)

We'll use the whole buffer as a range as there's no other examples like this one later on.
Remember for global, you can leave off the range if it's the whole buffer.

The command to uppercase a line is `gUU` (short for `gUgU` from [kata 6](006_operation_shortcuts.md)),
so we'll use a normal command.

Putting that together:

```
:global /^\V<start>/+1, /^\V<end>/-1  normal gUU
```

(I spaced it out to make it easier to read, but you don't need the spacing)

```
<start>
Boban
<end>

Don't upper case me!

<start>
Bobanita
<end>

Nor me!

<start>
Bobanish
And friends
<end>
```

# Exercises

## Exercise 1

Comment out the python code in the blocks below.

These are the only python blocks in the file so we can use the whole buffer for our range.

Remember the general form is:

```
[RANGE] global /[START PATTERN]/[OFFSET], /[END PATTERN]/[OFFSET] [EX COMMAND]
```

Run this command:

```
:global /^```python$/+1, /^```$/-1  normal I#
```

```python
a = None
b = 1
```

```python
c = a
def foo():
    return None
```

## Exercise 2

Sort the sets below (the blocks inside curly braces),

But don't sort the arrays (the blocks inside square brackets) as ordering matters for arrays.

Recall the general form:

```
[RANGE] global /[START PATTERN]/[OFFSET], /[END PATTERN]/[OFFSET] [EX COMMAND]
```

Put your cursor on the command below and run it:

```
:.,+30 global /\v^\{$/+1, /\v^\}$/-1 sort`
```

```
{
  "margarita",
  "boban",
  "james",
  "hayworth",
}

[
  "zij",
  "thilonator",
]

{
  "bobanita",
  "rohan",
  "aelfric",
}

[
  "valentine",
  "adil",
]
```

The searches are `\v^\{$` and `\v^\}$` respectively. Breaking those down:

- `\v` - very magic mode (which makes `^` and `$` magic, ie. anchors)
- `^` - start of line
- `\{` or `\}` - literal curly (escaping it just in case it's magic as braces are in some regex languages)
- `$` - end of line

`sort` is a built in ex command.

# The lie

Above we described this as another "general form" for the global command. Here it is with one of our examples.

```
[RANGE] global /[START PATTERN]/[OFFSET], /[END PATTERN]/[OFFSET] [EX COMMAND]

.,+30 global /\v^\{$/+1, /\v^\}$/-1 sort`
```

This is actually a lie - the above is _actually_ just the original global command form we learnt from kata 29.

It's hard to believe, but we'll break it down bit by bit to see what's really happening.

Below is the original form and below it shows the example in terms of the original form:
Recall the original form:

```
[RANGE] global /[PATTERN]/[EX COMMAND]

.,+30 global /\v^\{$/    +1,/\v^\}$/-1 sort`
-----         ------     ------------------
RANGE        PATTERN         EX COMMAND
```

Notice the extra spacing before the `+1` to emphasize where the pattern stops
and the ex command starts.

The part that will be confusing is the ex command itself. We'll break it down further:

```
+1,/\v^\}$/-1   sort
-------------   ----
  RANGE        COMMAND
```

Remember that some ex commands like `print`, `delete` and `sort` take ranges too.
In all the global examples from previous kata the range was always left out and would usually
conveniently be just the current line being processed by the global command which made sense in those contexts.

The text `+1,/\v^\}$/-1` is actually a range:

```
+1       ,      /\v^\}$/-1
-----    -      ----------
START  DELIM        END
```

The start is `+1` which is really `.+1`, ie. "the current line + 1".
In the context when that's executed, the current line is the opening brace matched by the global command.
So we're starting a range from the line below the open brace.

Then there's a comma, then the cryptic end part of the range `/\v^\}$/-1`.

What we haven't covered yet is that you can _use searches inside ranges_.

For example the range: `.,/boban/` defines a range from the current line to the next line containing "boban"
from the current line.

So breaking this down more:

```
/\v^\}$/  -1
--------  ---
NEXT '}'  OFFSET
```

Just like we can do `.-1` to mean "the current line minus 1", more generally you can use offsets on other kinds of markers:

- `$-3` - three lines up from the bottom of the file
- `'a+10` - ten lines after mark `a`
- `/boban/-2` - two lines above the next "boban"

So the end of our range means: "one line above the next closing curly".

Zooming out:

```
.,+30 global /\v^\{$/    +1  ,  /\v^\}$/  -1     sort`
-----        --------    ----------------------------
RANGE        PATTERN            EX COMMAND
FOR
global                   -------------------
                           RANGE FOR sort

                         +1  ,  /\v^\}$/  -1
                         --     ------------
                         .+1    next '}'  up
                         one              one
                         line             line
                         below
                         the '}'
```

So this idea of there being a start and end pattern and offsets is really a big lie.
But it's a very useful lie for transforming blocks of code.

If you do `:help global` you'll find no mention of this form there, it's just a convenient syntactic trick
that happens to work based on how ranges work.

The `+1` above isn't really attached to the first pattern.
In fact you can leave it out and this lie still works because ranges that are missing a left part fill it with `.`,
ie. `,3` is the same as `.,3`.
Likewise if you omit the second offset it has the effect of "offset=0" simply because you're not adding a relative
jump to the search for the end line.

# Conclusion

This second form of the global command is not technically part of vim, it's just syntactic trickery.

But it still works and you might as well think of it as another form for practical purposes because
it can solve a lot of problems for you.
