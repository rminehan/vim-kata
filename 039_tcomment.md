# tcomment

tcomment is a plugin which defines "comment" operators for commenting/uncommenting source code.

You can use it with existing line based text objects for commenting out chunks of code -
another win for operator + text object!

The plugin is [here](https://github.com/tomtom/tcomment_vim).

# The file type

You might be thinking: "Different languages have different ways of commenting, e.g.

```
// Java, Scala, C, C++
# Python, Bash
-- Sql
<!-- html, md -->
```

so how can tcomment know which one to use?"

Vim has a concept of "file types" which is standard information associated with a language (like syntax highlighting and commenting logic).

For example when a C file is opened, vim loads the C file type information and uses that to configure syntax highlighting and other editor operations.

tcomment also uses this file type information. You can see what it is using `filetype` or `ft` for lazy vimmers, e.g. `:set ft?<enter>`

Some file types come with vim (e.g. c) and some would require plugins (e.g. scala, markdown).

# Exercises

Through these exercises we'll be messing with the filetype to change the logic for commenting.
This will make the markdown syntax highlighting break.
To change the filetype back to markdown do `set ft=markdown` (or just restart vim if that doesn't work).

We've restricted the language examples to common languages like C or python as it's most likely they'll work out of the box without needing plugins.

## Exercise 1

In this exercise we'll introduce at the `gc` operator which toggles comments.

- set the filetype to python: `set ft=python`
- put your cursor anywhere on the text below
- do `gcip` to comment it (that is `gc` operator + `ip` text object)
- repeat the above to uncomment it
- hammer `.` a few times to continue toggling comments

```python

bar(b)
baz(b)

```

## Exercise 2

Often you just want to comment/uncomment the current line.

Similar to how `yy` yanks the current line, `gc` comes with a line based form `gcc`.

- set the filetype to C: `set ft=c`
- put your cursor on either line below
- do `gcc`
- hammer `.` a few times to continue toggling comments

```c

/* int a = 1 */
int b = 2

```

Note here that vim uses block comments: `/* .... */` rather than individual line comments: `// ...`.

## Exercise 3

When you use `gc` on a multi-line block, it's possible there is a mixture of commented and uncommented lines,
so some logic is required to determine the commenting action to take.

Typically it uses the logic:

> If all lines are commented, uncomment the whole block.
> Otherwise comment the whole block (potentially "double commenting" some lines)

- set the filetype to python: `set ft=python`
- put your cursor somewhere in the python block below
- do `gcip`
- (note it added comments because it could see a couple of uncommented lines)
- do `gcip` again or `.`
- (note this time it uncomments because all the lines were commented)

```python

# foo(1, 2, 3)
bar(4, 5, 6)
baz(7, 8, 9)

```

The logic above means that `gcip` inverted itself, ie. the second application undid the effect of the first application.

If vim had used more granular logic, (like only commenting/uncommenting certain lines), then we wouldn't get this effect.

## Exercise 4

If the commenting logic from exercise 3 doesn't do what you want, there are explicit commenting and uncommenting operators:

- `g>` comment everything
- `g<` uncomment everything

Let's use them below to add many layers of commenting and then remove those layers:

- set the filetype to python: `set ft=python`
- put your cursor anywhere on the block below
- do `g>ip` to increase the commenting
- hammer `.` a few times and note how the indentation keeps increasing
- do `g<ip` to decrease the commenting
- hammer `.` until all lines are uncommented

```python

# foo(1, 2, 3)
bar(4, 5, 6)
baz(7, 8, 9)

```

The paragraph was initially mixed, but after enough applications of `g<` that mixed style was lost.
This is something that `gc` won't do.

The only use case I can think of for these explicit operators is when you have a mixed block and you want to uncomment the commented lines
and not modify the uncommented lines.

For example you might want to uncomment some commented out debugging commands:

```

command1()
command2()
# print(foo)
command3()
# print(bar)
command4()

```

`g<ip` above would leave everything uncommented.

## (Optional) Exercise 5

In [kata 37](037_indentation_text_objects.md) we introduced the indentation text objects.

Often you're wanting to comment out code at a particular level of indentation (e.g. to comment out an `if` block).
We'll combine the `gc` comment operator with the `ii` text object to show the power of the "operator + text object" form.

(Note this assumes you've got the indentation text object installed)

Let's comment out the body of the `foo` function below:

- set the filetype to python: `set ft=python`
- put your cursor anywhere on one of the `line*` lines below
- do `gcii` (that is `gc` + `ii`)
- hammer `.` a few times to continue toggling comments

```
def foo():
  line1
  line2

  # line3
  line4

  line5
  # line 6
```

The indentation text object was very powerful here as it crossed the blank lines inside the method
and if you ran it from line1 or line2, it wouldn't have included the function signature.

If we also wanted to include the function signature, we could have used `gcai` from any of the `line*` lines.
(`ai` is also from the indentation plugin and it includes the top parent line for the indent block)

Note though that the dot operator wouldn't have reversed the commenting for `ai`.
Instead it would have considered the backtick line as being part of the indentation block.
Try it and see what happens!

# Conclusion

`tcomment` comes with 3 operators:

- `gc` - toggle comments
- `g>` - comment
- `g<` - uncomment

Vim shows its power by letting us apply these new operators with our existing text objects.

Most of the time `gc` is all you need.

In the case that your text object has multiple lines, it will:

- uncomment the block if all lines are commented
- comment the block otherwise (potentially leading to double commented lines)

When that behavior isn't what you want, then the `g>` and `g<` operators can be used, (but that's not very often).

Use `gcc` for toggling commenting on a single line.
Note that `g>` and `g<` don't come with a doubling operator but this is fine because we only need them in multi-line contexts anyway.

The logic for how to comment a line comes from the file type information vim has.
As more file types get added to vim, `tcomment` should be able to figure out how to incorporate those automatically.

Happy commenting!
