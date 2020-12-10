# Search magic

Previously we looked at buffer search and how the search term is a regex.

Most programming languages languages make heavy use of operators like `+.[]*<>` as part of their syntax.

However these characters also have special meaning in regexes, e.g. `+` means "one or more" and `.` means "any character".

It gets more confusing when you have a search that requires a special character in both it's literal form as well as its regex form,
e.g. "one of more `+`'s".
You will need to escape one of the plus's and not escape the other. Which way around the escaping happens depends on your search mode.

Vim has 4 search modes:

- `\v` very magic
- `\m` magic (default)
- `\M` no magic
- `\V` very no magic

So they are ordered from most to least magic-ness.

"Magic-ness" translates to characters having their regex meaning by default,
and escaping is required to give them their literal meaning.

In very magic mode, `a+` means "one of more a's" and `a\+` means "a followed by plus".

In very no magic mode it switches, ie. `a+` means "a followed by plus" and `a\+` means "one or more a's".

Very magic mode is close to perl's "extended" regex syntax which people usually treat as "standard" regex.
Magic is closer to standard old school regex (e.g. grep).

# 4 modes?

The reality is that it's too hard to remember the nuances of the two middle modes. Some things are special and some aren't.

It's much easier to use either very magic mode where everything has special meaning, or very no magic where nothing does.

So I wouldn't recommend spending time trying to memorize the two middle modes.

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

## Exercise 1

Search for the literal text "a+b".

The default search mode is "magic" where `+` and `.` are literal.

(I think) part of the reason for making this the default is because vim was originally written with C coding in mind and this makes
it easy to search for expressions like "a+b".

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

# Setting very magic to be the default

Some people want buffer search to default to very magic, so they remap `/` to `/\v` meaning that
when they type `/` from normal mode, it automatically inserts a `\v` at the start of their pattern.

# Summary

The various magic modes are about controlling whether special characters like `+.*` carry their regex meaning or literal meaning.

When your regexes get tricky, focus on just using either very magic (`\v`) or very no magic (`\V`).

That will make it simple and consistent as to which characters you need to escape.

# Further reading

[Vim wiki article](https://vim.fandom.com/wiki/Simplifying_regular_expressions_using_magic_and_no-magic)
