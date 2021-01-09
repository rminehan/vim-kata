# Search text object

Today's kata looks at the `gn`/`gN` text objects which represent the next/previous match for your last search.

For example if you just searched for "boban", then `gn` represents the next match for that in your buffer
from your cursor onwards.

# Syntax

## operator + text object

Like usual, we can combine `gn` with our existing operators.

For example supposed our last search was "boban" and we did `cgnbobanita<escape>`:

- `c` - change operator
- `gn` - next match for "boban"
- (this will delete it and put you in insert mode where the match was)
- `bobanita<escape>` - replace it with "bobanita" and return to normal mode

## visual selection

`gn` is a bit confusing in that if you want to visually select the next match,
rather than doing `vgn`, you just do `gn`.

Doing `vgn` will put you insert visual mode and then extend the selection to the end of
the next search match (where `n` would extend you to the front of the match).

This might feel inconsistent because you're using to doing say `vip` for "select paragraph"
or `vae` for selecting the whole buffer or `val` for selecting the current line.

Keep in mind though, those are cases where your cursor is already inside the text object you're targeting.

Remember as well that `vip` is two distinct commands: `v` for visual mode and `ip` which extends the current
visual mode to include the current paragraph. They are not one command the way `dip` is.

Doing `vgn` is really `v` + `gn`, ie. "Go into visual mode, then extend the selection to the next match".

So the visual mode behavior actually makes sense and what's more ad-hoc is that there's a command `gn`
that puts you into visual mode selecting the `gn` text object.

# Do we need this?

We already have `n` and `N` to navigate us to the next match, so do we need `gn`.

For example to replace the next "boban" with "bobanita" I can just do `n` to jump to that match then `ciwbobanita<escape>`:

- `c` - change
- `iw` - inside word
- `bobanita<escape>`

What if you'd matched the "boban" inside of "mcboban" though? That `ciw` would have deleted the "mc" where the `gn`
text object is specifically for the "boban" part and would preserve the "mc".

Similarly what if the pattern is really complex and matches something that doesn't nicely correspond
to an existing text object like `iw` or `iW`? For example a pattern for matching email addresses.
Or maybe some matches are simple and some are complex and require different change mechanisms. 
It's much cleaner to have a text object that represents exactly what was matched and a change you can apply
universally to your matches with the `.` operator.

Then you might reply: We can do this with the interactive `substitute` command, e.g. `%s/boban/bobanita/gc`.
That's true, but there might be operators we want to apply to the match that can't be represented by the
substitute command on a complex regex pattern (or it's tedious to do it),
for example uppercasing, lowercasing, toggling case, surrounding with tags (ala vim-surround).
And we might not want to have to go cycle through all the matches, just nearby ones.

So whilst `gn` doubles up with other functionality we've already covered in some way,
I would flip it around and say you should view `gn` as the "default" as it lets us use "operator + text object" style
and use other methods like `substitute` only when `gn` can't do the job well (e.g. mass replacements).

Then we get the usual benefits of operator + text object:

- keeps the mental model simpler and more consistent
- it works well with the dot operator
- it's portable across buffers
- it's more extensible as we can use it with new operators

A minor point in favor of `gn` is that it works better with multiline searches.
See exercise 3 of [kata 49](049_advanced_regex_7_multiline.md).

Finally another little pro for `gn` vs using `n` is that it removes the ambiguity related to search direction.
When you hit `n`, it follows the search direction of your last search (forwards for `/` and backwards for `?`).
Chances are you won't be able to remember which direction you used.
The nice thing about `gn` is that it's _always_ forwards from your cursor and `gN` is always backwards.

# Exercises

For these exercises we'll be searching our old friend Boban.
Put "boban" into the search register by doing either:

- `:let @/ = 'boban'`
- `/boban<enter>`

## Exercise 1

Replace the "boban"s below with "bobanita"s.

- put your cursor on the opening backticks
- do `cgnbobanita<escape>`
- hammer on `.` until they're all converted

```
Her name was boban, she lived in the distant Albanian mountains.

A courier came to boban's door with a dinner invitation.

But before boban could read it, the courier was taken.
```

Note an important part of this working is that the cursor is at the end
of each "bobanita" (not the start) after each edit.
"boban" is a substring of "bobanita" so if the cursor ended at the front of the
string we'd end up being stuck on the same replacement.

## Exercise 2

Uppercase all the boban's in the block below. Don't uppercase false positive matches like "bobanita" and "mcboban".

Note: Usually we'd deliberately put word boundaries into the search term to reduce false positives,
but the point of this kata is to get your skipping matches and do the equivalent of an interactive `substitute`.

Also make sure you've got case insensitivy switched on so that `/boban` matches "Boban" (`set ignorecase`).

- put your cursor on the starting backticks
- do `gUgn` to uppercase the first match (which luckily is a true positive)
- do `n` to just to the next supposed match (or if your search direction is backwards, use `N`)
- if it's a true positive, do `.` otherwise do nothing
- continue using `n` with/without `.`
- once you get to the end, put your cursor on the end backticks
- do `gugN` (to lowercase the last match which luckily is a true positive)
- use `N` with `.` to interactively undo all the uppercasing

```
boban said to himself: "I've been such a boban, why did I send a courier?"

Meanwhile Bobanita asked McBoban: "What do you think? Should I have a dinner date with boban? He can be a real boban..."

Old McBoban replied: "A boban is as a boban does. Such has always been the way of things..."
```

## Exercise 3

Uppercase all the characters from the first Boban to the second below inclusive of the Bobans.
This is an ad-hoc text object so visual selection makes sense herej

- put your cursor on the opening backticks
- do `gn` to visually select the first "Boban" 
- do `n` to extend the selection
- (whoops! That extends it over the 'B' but not the whole word)
- hit `h` a few times to move the anchor clearly to the left off "Boban"
- do `gn` to extend the selection over the whole of the second "Boban"
- do `gU` to uppercase the selection

```
And they all yelled Boban the mightiest of Boban very loudly
```

# Conclusion

`gn` is another useful text object that's bundled into vim.

We can use it to shift some more of our use cases from substitute to operator + text object.
