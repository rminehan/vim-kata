# Brackety text objects

Oh man these are so useful...

Vim contains built in text objects like:

- `i(` - inside round brackets
- `a(` - around round brackets
- `i{` - inside curly braces
- `a{` - around curly braces

We can combine these with our standard operators to do some really powerful refactoring.

The `i` text objects represent what's inside a set of brackets/braces/tags, but not the outer brackets/braces/tags.

The `a` text objects include the contents inside the brackets and the brackets themselves.

Let's unleash these bad boys...

# Exercises

## Exercise 1

Change the text inside of all brackets to "Enxhell".

- put your cursor somewhere inside the first brackets
- do `ci(Enxhell<escape>`, which is:
    - `c` - change
    - `i(` - inside brackets
    - (this will delete what's in brackets and put you into insert mode)
    - `Enxhell` - type the word "Enxhell"
    - `<escape>` - return to normal mode
- move your cursor to somewhere inside the next bracket
- invoke the power of the mighty dot (do `.`)
- repeat until all brackets are converted

```
There was a man (Boban),
he was odd (they called him Boban),
he spoke of ML (in a Bobanish way)
but people couldn't understand him (the Boban).
```

## Exercise 2

Change all the text in the curly braces to upper case.

This is much like the above, just using the `gU` operator instead of `c`.

- put your cursor somewhere inside the first braces
- do `gUi{`, which is:
    - `gU` - upper ase
    - `i{` - inside braces
- move your cursor to somewhere inside the next braces
- invoke the power of the mighty dot (do `.`)
- repeat until all braces are converted

```
There was a man {Boban},
he was odd {they called him Boban},
he spoke of ML {in a Bobanish way}
but people couldn't understand him {the Boban}.
```

## Exercise 3

Remove all the html tags from the text below.

This time we will use the text object `at`:

- `a` for "around" (everything - the inner contents and the tags themselves)
- `t` for html tags

So:

- put your cursor somewhere inside on first tag
- do `dat`, which is:
    - `d` - delete
    - `at` - around tags
- move your cursor to somewhere on the next tag
- `.`
- repeat until all tags are removed

```
There was a man<bold> Boban</bold>,
he was odd<bold> they called him Boban</bold>,
he spoke of ML<em> a Bobanish way</em>
but people couldn't understand him<div> the Boban</div>.
```

# There's many more

Standard vim also has these brackety text objects for:

- square brackets `[]`
- carets `<>`
- double quotes `""`
- single quotes `''`
- backticks

With some basic plugins you can get many more like this (e.g. "between pipes, between forward slashes").

There's even "sentence" text objects `is` and `as` for sentences delimited by '.'.

There are even more advanced ones for individual function arguments inside some round brackets separated by commas
which is useful for most languages (java, python, scala etc...)

# Smart jumping

With some handy plugins installed, you don't even need to have your cursor inside the brackets to use the text object.

For example:

```
foo(bar, baz, bobanTheMighty.name)
```

With the cursor at the start of the line, `ci(` will jump the cursor into the brackets, and then do `ci(` as before.

Likewise `cia` would change the first argument if invoked from the start of the line.

# Warning

Such unfettered editing potential may cause you to become delirious and drunk on power.
