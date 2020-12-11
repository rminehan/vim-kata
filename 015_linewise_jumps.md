# Linewise jumps

Sometimes you want to jump to a specific position on the current line.

Using buffer search for this is overkill:

- if you have settings like `incsearch` it will make your cursor jump around
- it will pollute your buffer with highlighting
- it will pollute your search history
- it uses regexes which causes confusion when you're wanting to jump to special characters

Instead vim has simple commands `f` and `t` for jumping to/before a character on the current line.

# Shortcuts

`f` and `t` define motions. The general usage is: `f[CHARACTER]` and `t[CHARACTER]`.

`f` will jump your cursor to the next instance of that character on the current line.

`t` will jump your cursor to _just before_ the next instance of that character on the current line.
(This is a lot more useful than it sounds - I probably use it more than `f`)

Often your line will have multiple instances of that character.
You can use `;` to jump to the next one, and `,` to just to the previous one (unless you've remapped it like me!)

There is also `F` and `T` which are analogous to the above but search backwards.

# Exercises

## Exercise 1

Jump to each dot in the line of text below.

- put your cursor at the start of the line
- do `f.`
- (this should jump you to the '.' after "He was")
- press `;` repeatedly to jump to subsequent dots

```
He was. He is. He'll always be. The Boban.
```

Note that dot here is a literal dot, not some regex for "any character". `f` and `t` are always for literal characters. 

## Exercise 2

Delete the text in brackets below (including the space just before it).

- put your cursor at the start of the line
- do `t(`
- (that should land you on the space before the opening bracket)
- do `D` to delete to the end of the line

```
How did you find him? (He's a sneaky Boban)
```

See, it _is_ useful! Doing that with `f` would have required a fiddly jump back one character.

## Exercise 3

Recall from [lesson 2](002_motions_as_text_objects.md) that motions can be used text objects.

Use this powerful knowledge to delete everything _before_ the bracketed text.

- put your cursor at the start of the line
- do `dt(` which is:
    - `d` operator
    - `t(` - a motion which gets converted to a text object meaning "from here until just before the next bracket"

```
How did you find him? (He's a sneaky Boban)
```

Note that if you did `df(` here it would be even more messy because you'd delete the first bracket and you'd need to:

- enter insert mode
- type a bracket
- leave insert mode

That's 3 more keystrokes, plus it will:

- pollute your undo history with an extra change
- set your last "change" to inserting a bracket (which makes the mighty `.` less mighty if you wanted to reuse this `df(` trick)
- pollute your insertion history and last insert register (which we'll encounter later)

## Exercise 4

Delete the text before the second bracketed text "(Hmmm...)".

We can delete up to the question mark. There's two question marks though and that annoying little space...

- put your cursor at the start of the line
- do `df?` to delete to the first question mark
- do `.` to repeat that action
- do `x` to delete the space

```
This one is more sneaky? - (very sneaky) how will we do it? (Hmmm...)
```

This is 3 deletes which puts 3 changes into the undo history.

As an alternative we could have done `dt(` then `.`. That's a little cleaner.

We could also have used visual mode as it's an ad-hoc text object we're targeting:

- put your cursor at the start of the line
- do `v` to enter visual mode
- do `f?` to extend it to the first '?'
- do `;` to extend it to the next '?'
- do `l` to extend it one character to the right
- do `d` to delete 

In the above you could have likewise used `t(` as your first motion then `;` which would have selected the area in two steps
without needing to use `l`.

# Sneak

There is a plugin "sneak" which turns `s` into a motion that takes _2_ literal characters and can search across multiple lines.

This is a very practical way to do quick jumps to a piece of text you can see.

Usually the first two characters of a word are rare enough within your buffer that you'll only need to hit `;` a few times to
get where you want to go.

Sneak shares the advantages of `t` and `f`, e.g.

- it's good for targeting special characters like `+` and `.` without having to worry about regex escaping
- it doesn't pollute your search history

Sneak replaces a lot of quick searches you'd usually do with buffer search.

# Strategic jumping and quick scope

If you use `f` and `t` enough, you'll develop a nack for looking at the area you want to jump to and calculating which
character in that area is the most unusual (and hence most likely to get you there in one jump).

For example characters like `q` and `z` are less common in common (unless you're using ZIO) so if you see a `q` in the word
you want to jump to, then `fq` will probably get you there pretty quickly.

There is in fact a plugin `unblevable/quick-scope` designed to assist with this kind of thinking.

When you press `f` or `t`, it colors the letters in words that will get you to that word in one or two jumps.

This is a nice plugin because it doesn't actually modify the way vim shortcuts work,
it just assists you in using them in a more efficient way.

# Summary

`f` and `t` are very useful for doing quick jumps on the current line.

It's a motion which means you can convert it into handy text objects for our trusty "operator + text object" form.

`t` is surprisingly useful so definitely get into the habit of using it rather than using `f` then over-correcting.

Also don't forget `F` and `T` which move backwards.
