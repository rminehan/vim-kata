# Advanced regex exercise setup

There's many advanced regex kata and they all work in the same kind of way.

This article is designed as a quick reference to look through before doing the exercises from any of these kata.

# Cheat sheet

Don't forget there is a [cheat sheet](advanced_vim_regex_cheat_sheet.md) designed to complement the kata.

Remember it only has the advanced vim specific stuff on it - you can find millions of basic regex cheat sheets online.

# Settings

Unless otherwise specified, the kata assume you've got these settings on:

```vim
set incsearch ignorecase smartcase inccommand=split hlsearch
```

## ignorecase and smartcase

As covered elsewhere, these make your pattern case insensitive when it has just lowercase characters,
then it switches to case sensitive when an uppercase character is included.

## incsearch

`incsearch` doesn't affect the _logic_ of your searches,
but from a teaching perspective it's necessary as it allows us to interactively build a pattern and see the effects
of each new keystroke.

We don't need to hit enter to lock in the search, but we can still see what would get matched if we did.

Usually the instructions will explicitly tell you to hit enter to lock in a search,
so if that's omitted you can just escape out of the search.

## inccommand

The setting `inccommand=split` causes a pop-up window to appear in a little split and preview your `substitute` command.

It's analogous to `incsearch` but for commands rather than searches.

## hlsearch

This highlights matches for us in the buffer as we interactively search.

Like `incsearch` it doesn't affect the logic of our searches, just the experience.

# Searching the text block

Most of the exercises revolve around advanced search features.

Usually the point of the exercise is to craft a search pattern and use it against some back-tick'd block of text
to get the right matches, e.g.

```
Search
Me
```

If you have `incsearch` on, typing in search terms can cause confusing jumps around your buffer if your search
happens to match something at a different point in the buffer.

To get around this, the instructions usually implicitly assume you have visually selected the search text
and you're including `%V` in your pattern.

`%V` limits the search to only finding matches in the most recent visual selection, which reduces the jumpy behavior.
`%V` is properly introduced in the first advanced search kata (43).

So when you see instructions like:

> prepare the text area
> highlight the text area as usual

what it means is, do a linewise visual selection of the text area being searched, then press escape.

For example for the block below you could do:

- put your cursor on 'a'
- do `V` to enter line visual selection
- do `jj` to extend the selection down
- press escape

```
a
b
c
```

## The same old search prefix

In this context, usually the first thing you'll do in an exercise after preparing the text area
is to start a search with `/`.

Almost always that's going to mean doing: `/\v%V` which is:

- `/` - start a buffer search
- `\v` - set the remainder of the search to be very magic
- `%V` - only accept matches that start in the most recent visual selection

If you get sick of typing this out again and again you might want to remap `/` to `/\v%V` from normal mode:

```
nnoremap / /\v%V
```

or you could map a different letter like `?`:

```
nnoremap ? /\v%V
```

The exercises will almost always use very magic mode as they are quite heavy with special characters like:

- `%V`
- `<>` (word boundaries)
- `@...` (look arounds)
- `{...}` (ranges)
- `()` (capture groups)

The extra cost of those two characters at the front quickly makes us for itself in readability.

## Cursor position

Less often the exercises will also suggest you put your cursor at a certain location.

Usually that cursor position has been chosen because the text it's on will be a match most of the way through
the pattern as it's interactively typed.

Again this is to avoid jumpy `incsearch`, but probably isn't necessary if the pattern is using `%V`.

## Splitting

In some rare cases, the exercises will search for something not near the instructions,
e.g. anchoring to the start and end of the file.

For those cases I'd recommend splitting your screen with `:split` or `:vsplit` before running the search so that one split
stays on the instructions.

You can close the split with `:q`. You can jump between splits with `<c-w><c-w>`.

## Non-linear pattern building

Some exercises will make you construct a pattern out of order - not from left to right.

For example to enter a pattern like: `/\v%V<(\d{3})-(\d{3})>` the exercises might take you through these steps:

- `/\v%V`
- `/\v%V()-()`
- `/\v%V(\d)-()`
- `/\v%V(\d{})-()`
- `/\v%V(\d{3})-()`
- `/\v%V(\d{3})-(\d{3})`
- `/\v%V<(\d{3})-(\d{3})>`

This is usually to make the pattern valid for as long as possible so that it will match something interactively.

Doing it this way makes you see more clearly what effect each new key stroke causes which is better for learning.

For example if you typed `(\d{3})` then once you hit the `(`, the pattern is invalid until the closing `)`
and you'll see no matches between.
You won't see the effect of adding `\d` and then `{}` after it - only the final version `\d{3}`.

# Possible bug with smartcase and %V

In my neovim 0.4.4 and vim 8, when I include `%V` in a pattern it seems to trigger smartcase to make the pattern case sensitive.

I'm guessing it's seeing the capital 'V' in `%V` and `smartcase` is engaging case sensitivity.

In cases where we're affected by this bug and we want case insensitivity, we'll put `\c` in our pattern.
We're allowed to put it anywhere but putting in near the front with `\v` gets it out of the way.

I've posted an issue [here](https://github.com/neovim/neovim/issues/13837).

# Help

It's time for you to start learning how to use vim's built in help.

Usually this is done by doing `:help [TOPIC]`.

A lot of the help for patterns can be accessed by doing `:help /[ATOM]` and entering the atom as it would appear
in magic mode, e.g. `%V` is `\%V` in magic mode, so you'd do `:help /\%V` to learn more.

It's also worth just skim reading the pattern.txt article from top to bottom.
You can read it in a browser [here](https://neovim.io/doc/user/pattern.html) or by doing `:help pattern-search`.

Throughout the articles I've included search terms for use with help.
