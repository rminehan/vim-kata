# Buffer search

Searching text is a big topic but it's essential for fast coding.

We'll start with the humble `/` which searches the current buffer.

# Shortcuts

From normal mode, press `/` to initiate a buffer search.

Down the bottom of the screen you'll see a prompt asking you to enter a search term.

The search term is a regex but for today we'll just use simple search terms like "Boban".

Press enter to lock in the search then use `n/N` to jump forward/backward through the matches.

(You can use `?` instead of `/` to make the search where `n/N` jumps you backward/forward through the matches)

You can use `:set nohlsearch` to remove left over highlighting.
There are plugins which will make `<c-l>` clear the highlighting,
or are even smart enough to automatically figure out when you no longer want to see highlighted matches.

# Special search settings

To make the exercises work consistently for everyone, we'll standardize on some settings.

Run the following:

`:set incsearch<enter>` 
Search interactively while you type the search term

`:set wrapscan<enter>`
Wrap around the document when your search hits the bottom/top

`:set ignorecase<enter>`
Make searches case insensitive generally, e.g. searching "boban" will match "Boban"

`:set smartcase<enter>`
Make searches with an upper case letter become case-sensitive.

You can do all of the above in a one liner:
`:set incsearch wrapscan ignorecase smartcase<enter>`

# Exercises

All exercises will relate to searching this text area:

```
Boban
boban
  bobanita
bobanish
the_boban
BOBAN
 boban.com
Boban
```

## Exercise 1

Search in such a way that all the bobans in the text area get matched, the clear the highlights:

- do `/` to start a buffer search
- do `boban<enter>` (all Bobans should highlight - even uppercase ones)
- hit `n` a few times to navigate through them
- do `:set nohlsearch` to clear the highlighting noise

Because the search term was all lower case characters, it was case-insensitive and matched "Boban" and "BOBAN".
See the `smartcase` setting above.

## Exercise 2

Search in such a way that only the capitalized "Boban"s get matched:

- do `/` to start a buffer search
- do `Boban<enter>` (only the "Boban"s should match)
- do `:set nohlsearch` to clear the highlighting noise

The search became case sensitive because an upper case letter was introduced into it.

Your next question will be: "What if we want just the lower case Bobans then?"

## Exercise 3

To make a search case-(in)sensitive, put this somewhere in your search:

- `\c` - case insensitive
- `\C` - case sensitive

Note: it can go anywhere in the search term. This is handy because you're often halfway through writing your search
when you realize you need it to be case (in)sensitive.

Note as well: these settings override built in settings like `ignorecase` and `smartcase`.

Search in such a way that only the lowercased "Boban"s get matched:

- do `/` to start a buffer search
- do `\Cboban<enter>`

What if we wanted pure "boban"s though (not "bobanita", "bobanish", "the_boban" etc...)

## (Bonus) exercise 4

Vim has special symbols for word boundaries, they are:

- `\<` - left boundary
- `\>` - right boundary

Search in such a way that only the lowercased "boban"s are found:

- do `?` to start a backwards buffer search
- do `\<boban\>` (don't press enter yet!)
- (oops - we forgot we need a case-sensitive search to weed out those shouty "BOBAN"s)
- do `\C<enter>`
- hit `n` a few times and notice how it's taking you through the results from bottom to top

This will match "boban" and the boban in "boban.com" which is good or bad depending on context.
If we were searching code and boban was a variable name then "boban.com" would be a good match.

# Summary

We're just scratching the surface of buffer searching - there's more to come:

- regexes
- vim's special extended regex tricks (we saw some today)
- tricks to make building searches faster

It's a big topic!
