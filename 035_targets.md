# Morrrrre text objects!

The next few kata are about extending our armory of text objects.

Recall right back from [kata 1](000_operations_and_text_objects.md) our general form: "operator + text object"
and the idea that operators are decoupled from text objects.
When we add a new text object we can use it with all the operators we already know, so we can learn a relatively
small number of operators and text objects, but get the multiplication of those lists in terms of editing abilities.

# The need for plugins

We're going beyond the built in text objects which requires standard plugins.

Hence these kata require you're using vim or nvim as the plugins are vimscript based.

By this point you're probably already on one of those editors because of our [surround kata](025_surround_part_1.md).

# targets.vim

Today we're going to look at the plugin [targets.vim](https://github.com/wellle/targets.vim).

Make sure it's installed!

## New text objects

It introduces some nifty text objects:

- `a` based ones for "argument" to a function, e.g. `ia` and `aa`
- more pairs (e.g. `,`, `:`, `|`)
- `I` and `A` variations for nested text objects, e.g. `I"` for inside quotes ignoring boundary spaces
- "next" and "last" based variations on existing text objects

# Seeking

In conventional vim, if you run someting like `di(` (delete inside brackets),
it will only work if you're inside some brackets.

targets.vim modifies these pair based text objects such that if you're not inside the pair being targeted,
it will seek across the line to find the next pair and apply your change there.

This lets you jump to the pair and apply your change in one step which is very handy.

You can exploit this to merely jump inside a pair without changing anything by
doing `y` + text object, but this has the downside of polluting the anonymous and `0` registers.

targets also adds "next" and "last" based operators which explicitly use seeking, e.g.

- `cin(` is "change inside next round brackets"
- `dil[` is "delete inside last square bracket"

# Counting outwards 

Another change targets made to standard text object like `i(` is the ability to add counts for how many
levels of nesting you want.

For example: `d2a)` means:

> from the cursor, zoom out until you hit the second pair of round brackets, then delete it"

Example:

```
# Initially
outside ( middle ( inside ) middle ) outside
                    ^ cursor
d2a)

# Result
outside  outside
```

This is nice, but we could have also achieved this by leaning on our trusty dot operator: `da).`

For deleting _inside_ brackets though, dot wouldn't delete you to the next level because you're still
inside the original brackets.

# Big in and around

We've seen `a*` and `i*` style text objects where the `a` means "around" and `i` means "inside".

For example with pair based text objects, "around" means to include the pair itself:

```
this(boban)says
       ^ cursor

# Result of `di)`
this()says

# Result of `da)`
thissays
```

With word and line based text objects, "around" and "inside" usually refers to the whitespace around the
text object.

```
hello my friend
      ^ cursor

# Result of `diw` (two spaces)
hello  friend

# Result of `daw` (one space)
hello friend
```

targets.vim adds `I` and `A` which informally mean "even more inside" and "even more around" for pair based text objects.

```
this ( boban ) says
       ^ cursor

# Result of `dI)`
this (  ) says

# Result of `dA)`
this says
```

In the `dI)` example, inner whitespace touching the brackets was kept.

In the `dA)` example, trailing whitspace after close bracket was also deleted
(but not the whitespace before the opening bracket).

So `A` here is like `a` for pairs plus `a` for lines/words.

# Exercises

We can't cover everything in targets.vim, we'll just play with some of it.

## Exercise 1 - argument text object

There is a function being called below and we want to change the arguments being passed in:

- "boban " + jones -> theEnxhell
- 21 -> 16
- friends.map(...) -> otherFriends

This is a change to use our `*a` text objects where `a` means "argument".

- put your cursor somewhere at the start of the line
- do `cia` (change inside argument) which should delete the whole argument and put you into insert mode
  - note how it used seeking to find the first argument on that line
- type out `theEnxhell<escape>`
- do `cina` (change inside next argument) then ` 16<escape>`
- then `cina` (change inside next argument) then ` otherFriends<escape>`

```scala
val boban = makePerson("boban " + jones, 21, friends.map(makeBobanFriend(_, Singapore)))
```

Note that we could have used tricks like moving our cursor to the start of an argument then do `ct,`
(change to the next comma) to empty the arg, but this trick fails on the last argument because it's complex
and contains a function call itself.
Such an approach is also fiddly as you need your cursor at the right starting position and doesn't support seeking.

So the argument text object is a nice tool which doesn't require much thought and will generally work the right way.

## Exercise 2

As mentioned above, targets.vim adds a lot more pair based text objects, e.g. `i:` for "inside colons".

On each line below, put your cursor somewhere on "boban" and follow the instructions to the right: 

```
20 + 30 + boban + 40         di+
20 + 30 + boban + 40         dI+
20 + 30 + boban + 40         da+
www.boban.leadiq.com         da.
/home/boban/ml               ci/enxhell
|james|boban|clement|        ya|:echo @"
```

Note that for these operators the "around" versions (e.g. `a.`) only include the leading character.
For example in the last example you would have seen "|boban" get printed.
Only including one of the operators makes sense because only one of them would be associated with the inner argument,
whereas with brackets, the two bracket characters are a true pair.

## Exercise 3

We'll practice the nested or "multi" text operators.

On each line below, out your cursor somewhere on "boban" and follow the instructions to the right.

```
f( ( boban ) ) g            d2i)
f( ( boban ) ) g            d2a)
f( ( boban ) ) g            d2I)
f( ( ( boban ) ), moban )   d3i)
f( ( ( boban ) ), moban )   di)..  (won't work like the above)
f( ( ( boban ) ), moban )   d3a)
f( ( ( boban ) ), moban )   da)..  (this one works like above because it removes brackets)
```

# Summary

targets.vim is a big plugin with a lot of concepts.
On the first pass through it's unlikely you'll absorb all of it.

The main takeaways:

- many existing text objects have seeking added meaning you don't have to be "on" the text object to use it
- argument text objects are added: `*a`
- many operators like `+` and `.` now have "between" text objects
- `I` and `A` have been added for many text objects for better whitespace control

Next kata we'll add some more more custom text objects to our armoury.
