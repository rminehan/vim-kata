# vim-surround (part 1)

Vim surround is a much loved plugin that appears on almost all "top 10" plugin lists for vim.

It's made by Tim Pope and is something I used heavily.

# Installing vim-surround

This is the first kata where I've broken the rule that the kata should work out of the box on a minimal vim.

We're far enough into our kata now that anyone still hanging on would be ready to embrace some plugins and customization.

To do this kata you'll need either:

-  vim + vim-surround plugin
- nvim + vim-surround plugin
- spacemacs

The plugin is [here](https://github.com/tpope/vim-surround).

I recommend using [vim-plugged](https://github.com/junegunn/vim-plug) as a plugin manager.
It's quick to install and makes installing plugins very easy - particularly if you put your config under version control.

If you've gone to the effort of installing a plugin manager, also be sure to install vim-repeat.

The plugin is [here](https://github.com/tpope/vim-repeat) and it allows surround changes to
be repeatable with the mighty `.` (and probably makes `u` undo it in one step).

# What is vim-surround

It adds a new text object to vim - the "surround" text object -
this corresponds to the text surrounding something on both sides.

We can use this text object with `d` and `c`.

It's more abstract than the typical text objects we're used to and doesn't quite fit the operator+text object form,
but you'll get the hang of it pretty quickly.

# Exercises

Explaining how it works is too hard - much easier to dive into examples:

## Exercise 1 - removing surrounds

This exercise is all about deleting surrounding objects.

On each line, put your cursor insides the brackety thing, and follow the command on the far right side of the line.

```
(Boban)                   ds)
(Bobanita)                ds) or . if you have vim-repeat
[ The biggest Boban ]     ds]
(( Boban ))               ds) ds) or .
{[( A nested Boban )]}    ds) ds] ds}  (try it in different order)
<em>Boban!</em>           dst
```

Here `d` is for delete, then the next part is a text object:

- `s)` - "the round brackets surrounding the cursor"
- `s]` - "the square brackets surrounding the cursor"
- `st` - "the html tags surrounding the cursor"

From where the cursor is, it looks left and right until it finds the open-close pair matching the text object specified.
Then it deletes them.
So it's like a discontiguous text object.

## Exercise 2 - changing surrounds

Each line for this exercise corresponds to _changing_ a surround, ie. replacing single quotes from single quotes.

As above, put your cursor inside the brackety thing and follow the command.

```
(Boban)                   cs)]
(Bobanita)                cs)] or .
[ The biggest Boban ]     cs])
(( Boban ))               cs)} cs)} or .
{[( A nested Boban )]}    cs)<em> cs]<b> cs}<div>  (try it in different order)
<em>Boban!</em>           cst"
```

# Exercise 3 - changing with spaces

This exercise set is similar to exercise 2 except we're using "left sided" surrounds for the new surround.

```
(Boban)                   cs)[
(Bobanita)                cs)[ or .
[The biggest Boban]       cs](
(( Boban ))               cs({ cs)}
( Boban )                 cs()
```

What we're introducing here is that `(` and `)` represent slightly different things.

Left sided surrounds include an optional interior space whereas right sided ones don't.

In particular in the `(( Boban ))` example, the two actions were slightly different.

The first was: `cs({`:

- `(` - find the text object with round brackets (and a space if one is there)
- `{` - replace it with curly brackets and a space

After that the text becomes "({ Boban })". The second change was: `cs)}`:

- `)` - find the text object with round brackets (and no space)
- `}` - replace it with curly brackets (and no space)

Which changes it to "{{ Boban }}".

If the second change was `cs(}` it would still match the round brackets,
ie. it will include whitespace in the target text object if it's there, but it doesn't require it.

The last example "( Boban )" with `cs()` is a little trick to quickly remove surrounding.

## Exercise 4 - adding surrounds

This one is a bit harder to get the hang of.

There isn't an intuitive operator for adding net new things (for example `d` deletes and `c` changes, but neither creates ex nihilo).

Hence the `ys` operator is added for creating new surroundings.

Another difference here is that because we're adding a new surround, we don't yet know where it is.
In the case of deleting a surround, you just tell vim what kind of surround you want to delete and it finds it.
But when you want to _add_ a surround, that concept doesn't make sense - instead we need to specify a text object to surround.

The general syntax is: `ys[TEXT OBJECT][SURROUND]`.

For example: `ysiw)` means:

- `ys` - add surround
- `iw` - around the current little word
- `)` - add round brackets without spaces

As usual, apply the operators on each line below. For each line have your cursor at the start of the line.

```
Boban                               ysiw)
Boban the champion                  ysiw)
Boban-the-champion                  ysiW)  "big word"
Boban the humble?                   ys$)   The dollar motion captures the whole line
Boban he said                       ysiw"
(Boban) said                        ysi)"  `i)` is "inside brackets" - see kata 16
(Boban) said                        ysa)"  `a)` is "around brackets"
Boban                               ysiw<em>
```

# Resources

[Plugin page](https://github.com/tpope/vim-surround) -
this has a quick tutorial to give you the gist, but might leave you disoriented.

[Futurile tutorial](http://www.futurile.net/2016/03/19/vim-surround-plugin-tutorial/) -
goes through it in much more detail and has some helpful tables

# Replacing `s`

If you disable vim-surround or do `vim -u NONE` you'll find that there is already an `s` operator.

(1) If you press it in normal mode it deletes the character your cursor is on and puts you into insert mode. 

(2) If you have text visually selected and press `s`, it will delete it and put you into insert mode.

So vim-surround actually hijacks the `s` operator. Vim has already used all the common keys and the reasoning
was that `s` is not doing anything particularly unique:

(1) - this is the same as doing `cl` or if you want to stay in normal mode just do `r` then the new key

(2) - `c` behaves in exactly the same way in visual mode

So there was a Popal decree to repurpose `s`.

Note as well that the sneak plugin (mentioned earlier in [linewise jumps](015_linewise_jumps.md)) also repurposes `s`.

So sacrificing `s` gives us 2 useful plugins. 

It was for this reason that we never mentioned `s` in these kata and I've tried to always steer you towards using `c`
for "changy" changes so that you wouldn't have to unlearn `s` later on.

# Wrapping up

vim-surround takes a while to get into your muscle memory (particualrly `ys`), but it's well worth the effort.

Changing surrounds is very frequent and without vim-surround you have to make fiddly jumps
to the start and end of each surround.

vim-surround is also an interesting example of text objects going beyond the typical contiguous blocks we tend to think of.

There is more to go and we'll carry on with vim-surround next time.
