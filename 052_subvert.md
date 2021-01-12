# Subvert

Today we're covering the `Subvert` command from Tim Pope's awesome [abolish](https://github.com/tpope/vim-abolish).

It's a beefed up version of `substitute` which can do some common tasks which would involve you tying yourself
up in knots trying to use regular vim mechanisms.

# Motivation

Cast your mind all the back to [kata 14](014_substitute.md) when you were still just a baby vimmer,
nervous and intimidated by the world of vim.

Our task was to replace all the "Boban"s with "Enxhell"s below, with a some caveats:

- match case insensitively
- Bobn should also get replaced
- don't replace Bobanita

```
Boban is a plucky young Albanian, not like Bobanita.
She'd say: "That boban, he's a real card".
Boban would often say "mate" to sound Australian. That's so bobn!
Have you met Boban or Bobanita?
```

The search pattern we ended up using was:

```
/\c\v<boba?n>      .,+3   from the cursor down 3 lines (4 in total)
                   \c     case insensitive
                   \v     very magic
                   <..>   word boundaries (to keep out Bobanita)
                   boba?n matches "boban" and "bobn"
```

The replacement was always "Enxhell" which led to some discussion afterwards about issues.

Here's a list of issues:

- the original casing isn't preserved - a lowercase "boban" will become "Enxhell"
- nicknames like "bobn" won't get translated to the equivalent Enxhell nickname "junior"
- if we _were_ going to also match plural "Bobans", we'd want those to become plural "Enxhellens"

Now there _are_ ways to do this with regex, but it will get pretty painful.

The `Subvert` command is designed for exactly these kinds of operations.

# Subvert

The basic syntax is the same as `substitute`:

```
[RANGE] Subvert/[SEARCH]/[REPLACE]/ARGS
```

Like `substitute`, you can shorten it to just `S` (uppercase).

And omitting the range defaults it to the current line.

The info on the github readme is a pretty terse summary.
Once you've got it installed you can do `:help :Subvert` for more detailed info.

We'll jump straight to the exercises to see how it behaves differently to `substitute`.

# Exercises

To do today's exercises you'll need to install the [abolish plugin](https://github.com/tpope/vim-abolish).

Also do:

```vim
set ignorecase smartcase
```

## Exercise 1 - preserving case

Replace all the "Boban"s below with "Enxhell" but preserving case, e.g. "boban" -> "enxhell", "Boban" -> "Enxhell".

- visually select the below
- do `:Subvert/boban/enxhell/g<enter>`

```
Boban is a plucky young Albanian.
People would say: "That boban, he's a real card".
When he's with hip young people he spells his name BoBaN!
Boban would often say "mate" to sound Australian. That's so bobaN!
Have you met Boban? You'll hear him running around yelling BOBAN!
```

You should get:

```
Enxhell is a plucky young Albanian.
People would say: "That enxhell, he's a real card".
When he's with hip young people he spells his name BoBaN!
Enxhell would often say "mate" to sound Australian. That's so bobaN!
Have you met Enxhell? You'll hear him running around yelling ENXHELL!
```

So we can see it go a lot of them (but not all):

```
Boban -> Enxhell
boban -> enxhell
BOBAN -> ENXHELL
```

If you do hit `/` and `<c-f>` or do `:History/<enter>` you should see a previous search of:

```
\v\C%(BOBAN|Boban|boban)
```

so you can see that `Subvert` was looking for

- totally uppercased
- totally lowercased
- capitalized

versions of our search term.

It missed "BoBaN" and "bobaN" - they're too hip and millenial.
It makes sense though. What would "bobaN" become? "enxhEll" (5th letter) or "enxhelL" (last letter)?
These aren't common cases and it's not really clear how they should be handled.
No matter what you do, there'll be a young person who complains about it anyway.

All up it was a pretty simple command it did something very powerful without much fuss.
Note as well we didn't add some "preserve case" flag - it's just the default behavior.

## Exercise 2 - suffixes

Replace "Boban" with "Enxhell" below, but preserve plurals: "Bobans" -> "Enxhellen".

Also don't change "Bobanita"s. 

- visually select the text
- do `:S/Boban{,s}/Enxhell{,en}/g<enter>`

```
In the distance I saw an academia of Bobans approaching (that's the collective noun).
All the Bobans were reading academic papers and disagreeing with each other.
One boban said to the others, "BOBANS! This academic paper describes a process for writing academic papers
that can't create the original academic paper. Useless!"
Bobanita was there, but she wasn't impressed.
```

You should get:

```
In the distance I saw an academia of Enxhellen approaching (that's the collective noun).
All the Enxhellen were reading academic papers and disagreeing with each other.
One enxhell said to the others, "ENXHELLEN! This academic paper describes a process for writing academic papers
that can't create the original academic paper. Useless!"
Enxhellita was there, but she wasn't impressed.
```

The syntax used was:

```
SEARCH      REPLACE
Boban{,s}   Enxhell{,en}

Boban  -> Enxhell
Bobans -> Enxhellen
```

The `{...}` sections describe how to pair up matches to replacements.
In our case we had an empty string and "s" in the search, paired up to an empty string and "en" in the replacement.

The search was all the combinations of case with suffixes:

```
\v\C%(Bobans|BOBANS|bobans|BOBAN|Boban|boban)
```

Overall `Subvert` mostly worked here, except it changed "Bobanita" to "Enxhellita".
We didn't guard against this so it makes sense.

We'll fix this in the next exercise.

## Exercise 3 - match whole words

In the previous exercise we did:

```
'<,'>S/Boban{,s}/Enxhell{,en}/g
```

which:

- worked on "Boban" and "Bobans"  (good)
- worked on "Boban" in "Bobanita" (bad)

Can we add a word boundary `>` to protect from this kind of thing?

- visually select the text below
- do `:S/Boban{,s}\>/Enxhell{,en}/g<enter>`
    - hmm... got an error

```
boban
Boban
BOBAN
bobans
Bobans
BOBANS
McBoban
bobanita
```

The command will have given an error like:
 
```
E486: Pattern not found: \v\C%(BOBANS\\>|Bobans\\>|bobans\\>|Boban\\>|boban\\>|BOBAN\\>)
```

A few observations:

- it's taking our text `Boban{,s}\>` and transforming it into a complex pattern
- it's putting `\v` (very magic) at the beginning
- our `\>` word boundary became `\\>` which means literal backslash then right word boundary
    - that won't match as we don't have literal backslashes in our text
- maybe if we just leave off the `\` it will work?

So:

- do `gv`
- do `:S/Boban{,s}>/Enxhell{,en}/g<enter>`

This time we get:

```
E486: Pattern not found: \v\C%(\\V<BOBANS>|\\v<Bobans>|\\v<bobans>|\\v<boban>|\\V<BOBAN>|\\v<Boban>)
```

It's putting `\\v` or `\\V` before each potential match which means literal backslash then 'v' or 'V'.

My take away from this is that `Subvert` is not intended for use with magic characters.
There's complex text transformations and it's too hard to reason about what magic characters will end up as.

For this particular case though thankfully there's a nice answer: the `w` arg.

- visually select the text below
- do `:S/Boban{,s}/Enxhell{,en}/gw<enter>` (note the extra `w` arg)

```
boban
Boban
BOBAN
bobans
Bobans
BOBANS
McBoban
bobanita
```

You should get:

```
enxhell
Enxhell
ENXHELL
enxhellen
Enxhellen
ENXHELLEN
McBoban   <---- not replaced
bobanita  <---- not replaced
```

## Exercise 4 - just the shouty BOBANS

`Subvert` gives us both case preservation and a language for permutations.

What if we just want the latter, e.g. we want:

```
BOBAN  ->  ENXHELL
BOBANS ->  ENXHELLEN

# The rest don't change
boban  ->  boban
bobans ->  bobans
...
```

Thankfully there's a flag `I` which disables case variation:

- visually select the text below
- do `:S/BOBAN{,S}/ENXHELL{,en}/gwI<enter>`

```
One boban yelled: "BOBANS unite! I will no longer be a BOBAN!"
Bobanita yelled back: "BOBANITA not happy!"
```

You should get:

```
One boban yelled: "ENXHELLEN unite! I will no longer be a ENXHELL!"
Bobanita yelled back: "BOBANITA not happy!"
```

Note the "boban" didn't change to "enxhell".

## Exercise 5 - match inside variable names

Vim's concept of a "word boundary" is often not quite what we want with `w`. For example usually:

- "McBoban" is one word
- "Mc_Boban" is one word
- "Mc-Boban" is two words
- "Mc.Boban" is two words

To convince yourself of this, put your cursor at different points in the words and do `viw`.

Above I put "usually" because it comes back to how `\k` and `:iskeyword` are defined.
Usually `-` and `.` aren't considered keyword letters hence they split the words above.

To make the point clearer:

- visually select the text below
- do `:S/boban/enxhell/w<enter>`

```
boban
Boban
BOBAN
Mcboban
McBoban
McBOBAN
Ah_boban
Ah_Boban
Ah_BOBAN
Mc.boban
Mc.Boban
Mc.BOBAN
```

Some will change. The ones that don't change don't have the 'b' on a word boundary.

```
boban -> enxhell
Boban -> Enxhell
BOBAN -> ENXHELL
Mcboban
McBoban
McBOBAN
Ah_boban
Ah_Boban
Ah_BOBAN
Mc.boban -> Mc.enxhell
Mc.Boban -> Mc.Enxhell
Mc.BOBAN -> Mc.ENXHELL
```

What if we wanted to consider `_` (e.g "Ah_boban") and case change (e.g. "McBoban") as boundaries though?

`Subvert` has a `v` flag for this:

>  v: Match inside variable names (match my_box, myBox, but not mybox)

- visually select the text below
- do `:S/boban/enxhell/v`

```
boban
Boban
BOBAN
Mcboban
McBoban
McBOBAN
Ah_boban
Ah_Boban
Ah_BOBAN
Mc.boban
Mc.Boban
Mc.BOBAN
```

You should see every line change _except_ "Mcboban".

The search used for this is quite epic:

```
\v\C%(<|_@<=|[[:lower:]]@<=[[:upper:]]@=)%(BOBAN|Boban|boban)%(>|_@=|[[:lower:]]@<=[[:upper:]]@=)
```

For an even more epic search let's put back plurals:

- visually select the text below
- do `:S/boban{,s}/enxhell{,en}/v<enter>`

```
enxhell
Enxhellen
ENXHELL
Mcbobans
McEnxhell
McENXHELLEN
Ah_enxhell
Ah_Enxhellen
Ah_ENXHELL
Mc.enxhellen
Mc.Enxhell
Mc.ENXHELLEN
```

This one has search term:

```
\v\C%(<|_@<=|[[:lower:]]@<=[[:upper:]]@=)%(Bobans|BOBANS|bobans|BOBAN|Boban|boban)%(>|_@=|[[:lower:]]@<=[[:upper:]]@=)
```

## Exercise 6 - prefixes

There's nothing to stop us putting `{...}` sets before the main body of our match.

Let's change from possibly Spanish Bobans, to Singaporean Enxhells:

- visually select the text below
- do `:S/{,el}boban{s,}/{,ah}enxhell{,en}/w<enter>`

```
boban
Elboban
Bobanita
Elbobans
ELBOBANS
Bobans
```

You should get:

```
enxhell
Ahenxhell
Bobanita
Ahenxhellen
AHENXHELLEN
Enxhellen
```

## Exercise 7 - switching words

Swap "Boban" and "Enxhell" in the sentence below.

(Inspired by Tip 96 from Practical Vim, lets use `Subvert` to switch two words)

Using regex swapping words gets pretty ugly. We can clean it up with some vimscript but we generally want to avoid vimscript.

- visually select the text below
- do `:S/{Boban,Enxhell}/{Enxhell,Boban}/g<enter>`

```
That Boban is such a Enxhell
```

You an even do more complex cyclic permutations like:

```
:S/{Boban,Enxhell,Bobanita}/{Enxhell,Bobanita,Boban}/g
```

# Vimscript as an alternative

It turns out that you're allowed to use vimscript in the replace section of a substitution using `\=`.

Practical Vim mentions this in tips 92 and 95.
To borrow from tip 95, highlighting the below and doing: `:s/\v\d+/\=submatch(0)+1<enter>`:

```
1
2
50
```

would yield:

```
2
3
51
```

It's taking submatch 0 (the whole match) and then coercing it into a number through adding 1.

Tip 96 shows how to use vimscript to achieve the same swapping we did in the exercise above.

For our example the final version would be:

```vim
[RANGE]s/\v<(boban|enxhell)>/\={"boban": "enxhell", "enxhell": "boban"}[submatch(1)]/g
```

It's taking the matched text (either "boban" or "enxhell") from group 1, then doing a dictionary lookup with it
to determine the replacement text.

In this case we got away without needing a plugin, but it's quite complex and repetitious and it _doesn't_ preserve case.

# Summary

`Subvert` is very useful for a variety of use cases where `substitute` falls short.

Syntactically it's been designed to work the same as `substitute` which eases the learning curve.

Note though that you can't start putting complex magic characters into your pattern.
For most use cases though, the built in flags like `w` and `v` should do what you need.

# Going further

If you've installed abolish then you might as well also look at the `:Abolish` commands and the "coercion" transformations.

They all relate conceptually.
