# Join operators

A surprisingly common thing I find myself doing is joining two lines, e.g.

```
// Before
The stranger lifted his
hat, it was the one they call Boban-Fet

// After
The stranger lifted his hat, it was the one they call Boban-Fet
```

Vim has an operator to do this: `J` (and its cousin `gJ`)

There is also an ex command for doing joins.

# Join is "eager"

Other operators we've seen wait for a text object before they act, e.g. `d` and `c`.

However `J` will eagerly join the next line onto the current line without waiting for a text object.

This is probably because most of the time you just want to join the current line with the next - it's a pragmatic optimization.

If you want to join across multiple lines, you need to visually select them then do `J`.

# gJ ?

The regular `J` operator will add a space between the joined lines, e.g.

```
// Before
Enxhell reached for
his gun, but Boban-Fet was too fast

// After
Enxhell reached for his gun, but Boban-Fet was too fast
                   ^ 
              space added
```

`gJ` however will not add a space:

```
// Before
Enxhell staggered back and fe
ll into Clement's lap

// After
Enxhell staggered back and fell into Clement's lap
```

# joinspaces and nojoinspaces

Vim is descended from the early text editors of the 16th century.

Because of this, vim follows the antequated practice of having two spaces after a '.' to start a new sentence.

For example if you use join these lines, notice that vim will probably add _2_ spaces:

```
//Before
With his last breath, Enxhell whispered to Clement: "Tell...
Rohan... I'm sorry I became...
a python developer

// After
With his last breath, Enxhell said to Clement: "Tell...  Rohan... I'm sorry I became...  a python developer
                                                       ^^                              ^^
                                                      2 spaces...                     2 spaces
```

You can turn this off by doing `:set nojoinspaces`. I have this set in my vim config.

You can turn it back on by doing `:set joinspaces`.

# Exercises

## Exercise 1 - repeated joins

Join all the lines below into a single line:

- put your cursor anywhere on the top line
- do `J` to join the next line onto it
- repeat with `J` or `.` until all lines

```
Clement got into his
ship -
he was determined to go to Dagobah
to find master Rohan and
become a vim master
```

## Exercise 2 - join from visual mode

Join all the lines below into a single line:

- put your cursor anywhere in the paragraph
- do `vip` to highlight the whole paragraph
- do `J` to join all the lines

```

Clement brought
an offering of
many tins of milo,
tim-tams and 
kangaroo burgers

```

## Exercise 3 - join with no spaces

Join all the lines below with no extra spaces:

- put your cursor on the first line
- do `gJ` to join the next line without adding spaces between
- repeat `gJ` or `.` until all lines are joined as one
- press `u` until all the lines are split back up
- do `vip` to highlight the paragraph
- do `gJ` to join all the lines in one go with no spaces between

```

"Code dadd
y you nee
d!" said the str
ange man hitting Cle
ment over the hea
d with a scala tex
tbook

```

# Summary

You might
not think it
but there
are many situations
where you need to
join lines of text
into a single line,
so keep
these up your sleeve
