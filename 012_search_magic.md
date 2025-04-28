# Search magic

Previously we looked at buffer search and how the search term is a regex.

Most programming languages make heavy use of operators like `+.[]*<>` as part of their syntax.

However these characters also have special meaning in regexes, e.g.

- `*` means "zero or more"
- `+` means "one or more"
- `.` means "any character"

It gets more confusing when you have a search that requires a special character
in both it's literal form as well as its regex form, e.g. "one or more `+`'s".
You will need to escape one of the pluses and not escape the other.
Which way around the escaping happens depends on your search mode.

Vim has 4 search modes:

- `\v` very magic
- `\m` magic (default)
- `\M` no magic
- `\V` very no magic

Above they are ordered from most to least magic-ness.

"Magic-ness" translates to characters having their regex meaning by default,
and escaping is required to give them their literal meaning.

e.g. in "very magic" mode, `a+` means "one of more a's" and `a\+` means "a followed by plus".

In "very no magic" mode it switches, ie. `a+` means "a followed by plus" and `a\+` means "one or more a's".

Very magic mode is close to perl's "extended" regex syntax which most people now think of as "standard" regex (e.g. egrep).
Magic is closer to what old schoolers used to consider "standard" regex (e.g. grep).

# 4 modes?

In "magic" and "no magic" mode some characters are magic and some aren't, e.g.

- `+` is literal
- `.` is magic

The reality is that it's too hard to remember the nuances of the two middle modes.

It's much easier to just use either:

- "very magic mode" where all those operators are magic
- "very no magic" where all those operators are literal

# Using a mode

Adding flags like `\v` or `\V` will make your pattern follow those semantics for _subsequent_ characters.

e.g. `\va+` means your searching in very magic mode (and plus means "one or more").

This means you can mix modes within a pattern,
e.g. `\Va ** \vb+` treats the `**` literally but the final `+` is a quantifier.
This would match "a ** bbb".

To emphasize the point, the flags only apply to characters after them.
This is different to `\c` and `\C` for case sensitivity, which can go anywhere in your pattern.

# Exercises

The exercises will be about searching this text area:

```
a+b

c*d

ab

a++

a+++

aa++

e.f
E.F
ekf
```

You might find it easier to split your window with `:split` so that the text above is always visible in one split
and the instructions are visible in the other.

In the last kata we set some search related flags.

Likewise for this one have case insensitivity and smart case set.

If you want to see your matches interactively have `incsearch` set.
It can be a little confusing though when it makes your cursor jump around whilst you type your search.

Having `hlsearch` enabled will be useful to see what matches your search.

The above paragraphs are summed up by a command like:

```
:set ignorecase smartcase incsearch hlsearch
```

## Exercise 1

Search for the literal text "a+b".

The default search mode is "magic" where `+` is literal and `.` is magic
(there's some discussion at the bottom about why this might be the default)

So we just do `/a+b`. This should match "a+b" but not "ab".

## Exercise 2

Search for one or more a's.

In magic mode (the default), we could do: `/aa*` or `/a\+`.

Or use very magic mode: `/\va+`

## Exercise 3

Search for the literal text "e.f" (case sensitive).

Remember that `\C` ensures it's case insensitive regardless of what settings you have (see previous kata).

The dot has its regex meaning in the default magic mode meaning `/\Ce.f` would pick up `ekf` which we don't want.

So we do either:

- `\Ce\.f` (magic mode)
- `\C\Ve.f` (very no magic mode)

## Exercise 4

Search for one or more a's followed by one or more plus signs.

This is probably most suitable for very magic mode: `/\va+\++`.

Breaking that down:

- `/` start search
- `\v` use very magic mode
- `a+` one or more a's (as plus has its regex meaning in very magic mode)
- `\+` literal plus
- `+` one of more (applied to a plus)

# Vim's seemingly odd default mode

As mentioned in exercise 1, the default mode is "magic" rather than "very magic" which is what you might expect.

My guesses at how this came about:

- when vi/vim was created, extended regex didn't exist yet or wasn't widely used/accepted.
  So at the time "magic" was made the default (and very magic might not have even existed).
  When extended regex did become popular, changing the default behavior in vim would have broken scripts
  and annoyed older curmudgeon-y vim users who are used to magic mode and like things the way they are.
  So new modes "very magic" and "very no magic" were added as a way to "opt out" but "magic" remained the default.

- early on a lot of vim users were C programmers and magic might make sense there for fast searching.
  For example `+` would occur a lot as a literal character in C, but `.` wouldn't.

## Setting very magic to be the default

Some people want buffer search to default to very magic.

Vim doesn't have a nice built in way to do this, but as a hack you can remap `/` to `/\v` so that everytime
you hit `/` from normal mode it's as if you typed `/\v` which populates your search with a `\v`.

Add this line to your vim config to get that change:

```
nnoremap / /\v
```

You can try this out in your vim session without making it permanent -
just run it as an ex command: `:nnoremap / /\v` then try a search!

You might also want to add an analogous mapping for `?` (backwards search) too!

# Summary

The various magic modes are about controlling whether special characters like `+.*` carry their regex meaning or literal meaning.

When your regexes get tricky, focus on just using either very magic (`\v`) or very no magic (`\V`).

That will make it simple and consistent as to which characters you need to escape.

# Further reading

[Vim wiki article](https://vim.fandom.com/wiki/Simplifying_regular_expressions_using_magic_and_no-magic)
