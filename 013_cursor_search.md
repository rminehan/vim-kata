# Cursor search

Sometimes you want to do a buffer search for a term that is already in your buffer.

For example you might be searching for the definition of a variable in some code you're reading.

Retyping the whole term can be tedious so vim gives a couple of ways that you can quickly get the text
under your cursor into a search term.

# Shortcuts

From normal mode, pressing `*` will cause vim to search for the little word under the cursor.
Note that it will use `<` and `>` as word boundaries.
You can use `g*` to have those word boundaries excluded.

Often this is good enough, however there might be times when you want to use what's under the cursor
as part of a larger search.

When you're typing out a search term (or any command actually), you can use:

- `<c-r><c-w>` to insert the little word under the cursor
- `<c-r><c-a>` to insert the big word under the cursor

(This kind of use of `<c-r>` is just a special case of a more powerful idiom, but we don't have time to cover that today)

# Exercises

These exercises will focus on searching for text in this block:

```
bobanthemighty

boban-the-mighty

boban

   bobanthemighty

bobanthemightyIII

the mighty boban

thebobanthemighty

bobanthemighty.name

bobanthemighty.job == None

thebobanthemighty.name

bigbobanthemighty.name

bobanthemighty_name
```

# Exercise 1

Search for all instances of "bobanthemighty". This is a search term that is long enough that writing it out by hand is annoying.

Put your cursor somewhere on the search term above then press `*`.

Note that it doesn't match "bobanthemightyIII" or "thebobanthemighty" because `*` adds in word boundaries at the start of your search.
You could change this by using `g*` instead.

You can see the previous search term by pressing `/` then the up arrow or `<c-p>` to dial up the last search.
Pro tip: You could also do `:echo @/`. This will make more sense later when we look at registers.

The search pattern should be `\<bobanthemighty\>`.
Here the `<` and `>` mark the word boundaries.
They are escaped because the vim ran the search in magic mode where they have literal meaning.

If you were going replicate this search by hand you'd probably use very magic mode so that you don't have to worry about remembering
whether `<` and `>` need escaping.
The search would be `\v<bobanthemighty>` (where `<` and `>` are treated as regex special characters).

# Exercise 2

We want to find instances of words starting with "bobanthemighty".

ie. we're hoping to match "bobanthemightyIII" but not "thebobanthemighty".

- turn off incremental search so that our cursor doesn't move whilst we type our search term: `set noincsearch`
- put your cursor somewhere on the search term
- press `/` to start the search
- do `\v` to enter very magic mode
- do `<` to mark a left word boundary
- do `<c-r><c-w>` to put "bobanthemighty" into the search
- do `<enter>` to complete the search

# Exercise 3

We want to find times where a call to `.name` has been made on a mighty boban, e.g.

- `bobanthemighty.name`
- `thebobanthemighty.name`

So:

- (again) turn off incremental search if you haven't already
- put your cursor somewhere on this search term: bobanthemighty.name (I've deliberately put it here without quotes or backticks)
- press `/` to start the search
- do `<c-r><c-a>` to copy the _big_ word under the cursor into your search
- do `<enter>` to complete the search

Whoops we also found `bobanthemighty_name`. This is because we were in magic mode where `.` has its regex meaning of "any character".

- dial up the last search by pressing `/` and `<c-p>` or the up arrow
- add `\V` somewhere before the `.` or escape it with a backslash
- do `<enter>` to complete the search

Note here that the match doesn't capture prefixes like "the" and "big".
If all you're wanting to do is jump to the line of the match this is fine.
If you're wanting to do some find/replace operation though then we'd need to improve our regex which will be tricky/awkward because
we're using a literal `.` in our search pattern.

# Exercise 4

We want to find calls to `.name` just for `bobanthemighty`. So we _don't_ want `thebobanthemighty.name` or `bigbobanthemighty.name`.

Here a left word boundary makes sense as in exercise 2.

- make sure incsearch is switched off
- have your cursor on this search: bobanthemighty.name
- do `/` to start a buffer search
- do `\v` to go into very magic mode (so that the next `<` will be regex)
- do `<` to signal a left word bounary
- do `\V` to go into very no magic mode (so that the dot will be literal)
- do `<c-r><c-a>` to insert the big word under the cursor 
- do `<enter>` to complete the search

Note in the above we used two magic modes. This is a bit convoluted and was more for teaching purposes.
In reality it would have made more sense to use `\V` and just escape the `<`.

# Visual star search

It's compulsory at this point to mention the visual star search plugin.

It lets you visually select a block of text and then press `*` to send that text into your search.

This is good for adhoc searches where the little word under the cursor is not specific enough and leads to many false positives. 

# Summary

`*` is a quick and dirty way to search what's under your cursor and usually it's good enough to find what you need with a few jumps. 

When you want to hand-craft more specific search patterns, you can use `<c-r><c-w>` (little word) and `<c-r><c-a>` (big word)
to mix the text under your cursor into your searches.

These two control shortcuts can be used for regular commands too, so it's handy trick to learn.
