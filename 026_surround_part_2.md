# vim-surround (part 2)

Last time we introduced vim-surround and 3 fundamental operations:

- deleting surrounds
- changing surrounds
- adding surrounds

Today we'll zoom in on adding surrounds and fill some miscellanous gaps.

# Exercises

## Exercise 1 - converting to function calls

When adding a surround, sometimes the reason for that addition is that you're actually passing that text into a function.

e.g. `30` -> `foo(30)`

vim-surround has a tool for surrounding with a function call - `f`.

The form is: `ys[TEXT OBJECT]f[FUNCTION NAME]` which surrounds your text object with a round bracket function call with
the function name specified.

Put your cursor at the start of each line and follow the instruction to the right:

```
30                ysiwffoo<enter>
40                ysiwfbar<enter>
"boban"           ysiWfbaz<enter>
x, y, z           ysfzfbaz<enter>
```

When you press `f` it should cause a prompt where you can track the new name you're typing.

The 4th one is a bit tricky, breaking it down: `ysfzfbaz<enter>`

- `ys` - add a new surround
- `fz` - the motion: jump to z (which creates a text object from the start of the line to the first 'z')
- `f` - function mode
- `baz<enter>` - entering the new function

## Exercise 2 - surrounding the current line

In [kata 6 - operation shortcuts](006_operation_shortcuts.md) we saw that many operators have a "shortcut mode"
where hitting the operator twice or using it in upper case mode was a quick way to get a text object related to
the current line.

Likewise with `ys`, repeating the `s` to form `yss` has the effect of making the current line the surround target.

Practice this with the examples below.
It doesn't matter where on the line your cursor is within each line as these are line based operations.

```
Type this `yss)` from normal mode
Type this `yss(` from normal mode
Type this `yss]` from normal mode
Type this `yss<b>` from normal mode
Type this `yss"` then `yssffoo` from normal mode
```

## Exercise 3 - surrounding and indenting

When adding surrounds, if you use `yS` (instead of `ys`) it causes the surrounded text to be indented to its own line.

You might have been expecting this to be a case like exercise 2 where it works on the current line.
That's not the case here - think of it instead as a modified surround operator.
In fact it has its own double operator `ySS` which means surround and indent the current line.

As usual, follow the instructions below to indent the surrounded text to its own line.
In each example, put your cursor at the start of the "Boban" before running the command.

```
The soldiers surrounded Boban the brave             ySiw<em>
They yelled out Boban come out!                     yS3W"
val x = Boban, the, brave                           yS3Wfyell
```

## Exercise 4 - visual mode

Lucky last is visual mode.

When you have an ad-hoc text object you want to surround, it's okay to use visual mode
(I won't call the police if there's no suitable existing text object).

Oddly, to add surrounds from visual mode, you use capital `S`, not little `s`.
I'm not sure why that's the case, perhaps Tim was wanting to allow regular `s` usage from visual mode.
Anyway we'll just roll with it for now.

In visual mode, the text object is implicitly the visually selected area, so we just need to specify the surround text.

The general form from visual mode is: `S[SURROUND]`.

In the examples below, put your cursor at the start of each "Boban" and follow the instructions on that line:

```
And he said: Boban is mighty            veeeS"
Boban (the (mighty) lisper)             vf);S)
list.mapBoban(_.toBoban)                veS]
```

I don't think there is a way to surround and indent on a new line from visual mode.
This is a quirk of vim-surround.

# Summary

These are a few extra tricks for adding surrounds to text objects.

If you find this too hard to remember, then just focus on the previous kata and really hammering in those shortcuts.
Later you can try again with these quirkier tricks.
