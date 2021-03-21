# Advanced regex 2 - looking around

Today's kata examines lookahead and lookbehind.

These are powerful regex tools for more refined searches.

Chances are you'll use this concept outside of vim too if you haven't encountered them already.

# Motivation

Sometimes you want to match a certain chunk of text, that has (or doesn't have) something before or after it.
But you don't want to include that thing in the match itself.

For example suppose we wanted to replace all boban's with enxhell's, except in the case of bobanita.

```
boban                REPLACE
boban junior         REPLACE
bobanita             DON'T REPLACE
old mcboban          REPLACE
ahboban              REPLACE
```

A pattern like `boban` is too permissive in that it will match the "boban" in "bobanita".

With our current tools, the best we can do is create a pattern like:

> "boban" + 3 characters that aren't "ita", e.g. `boban[^i][^t][^a]`.

This will consume 3 characters after our "boban" though. It will match the "boban ju" in "boban junior".

If we did a replace operation, we'd end up with "Enxhellnior" (a Spanish Enxhell) when
"Enxhell junior" is what was wanted (even though it's a truism).

We need a way to specify not matching something, but _without consuming characters_.

## Negative lookahead example

The solution here is a "negative lookahead". Visually select the block below then hit escape.

```
boban
boban junior
bobanita
old mcboban
ahboban
```

Then do:

```
/\v%Vboban(ita)@!
```

The `\v` is for readability to avoid having to escape `%V`, the round brackets and the `@`.
Recall the `%V` from the previous kata means to restrict the search to the last visually selected area.

The main part of the pattern is:

```
boban(ita)@!            boban - literal "boban"
-----     --            (ita) - literal "ita" grouped up into an atom
     -----                @!  - negative lookahead attached to the "(ita)"
```

Overall it means: "boban" not followed by "ita".

## Positive lookbehind example

Let's flip the problem around.

Suppose we want to replace all the boban's with enxhell's, but _only if_ they have "mc" or "ah" before them.

When we do the replacement we want to preserve the prefix, so we don't want the match to consume those characters.

Visually highlight the block below then hit escape.

```
boban
boban junior
bobanita
old mcboban
ahboban
```

Try:

```
/\v%V(mc|ah)@<=boban
```

It should match just the boban text in "mcboban" and "ahboban" above (not the "mc" and "ah" text itself though).

This is an example of a "positive" look behind. It means: "boban" with "mc" or "ah" before it.

It's different to the previous example in two aspects:

- it's looking backwards not forwards
- it expects the text to be there (not absent)

Lookahead and lookbehind are more general regex concepts not specific to vim,
but vim's way of expressing it probably won't be the same as other regex engines you come across.

# Exercises

See the [setup guide](advanced_regex_exercises_setup.md).

## Exercise 1 - positive look ahead

We'll search for boban's that have an alphabetical letter after them.

We can use `\a` ("alphabetical") to represent a letter. It's a shortened version of `[a-zA-Z]`.

- highlight the below block with linewise highlight then hit `<escape>`
- start a magic case insensitive search on the block with `/\v%V\c`
    - the `\c` relates to the potential bug pointed out last kata with `%V` triggering `smartcase`
- add `boban` (all the terms should light up)
- add `\a@=` (meaning "must have a letter after it)
    - `\a` - letter
    - `@` - look around operator attached to the previous atom `\a`
    - (no direction specified - defaults to forwards)
    - `=` - positive

```
boban          bobanita            mcboban            boban-jones    bobanJo
ahbobanita     BOBAN               Bobanson           ahboban
```

The final pattern should be: `/\v%V\cboban\a@=`

It should match the boban's from "bobanita", "ahbobanita", "Bobanson" and "bobanJo".

## Exercise 2 - a negative outlook

Search for lonely boban's with no alphabetical letters before or after.

We'll use a negative lookbehind and negative lookahead.

- highlight the below block with linewise highlight then hit `<escape>`
- start a magic case insensitive search on the block with `/\v%V\c`
- add `boban` (all the terms should light up)
- add `\a@!` (meaning "must not have a letter after it)
    - this should reduce the matches a lot but will still include errors like "mcboban"
- move your cursor back to just before the 'b' of "boban"
- add `\a@<!` (meaning "must not have a letter before it)

```
boban          bobanita            mcboban            boban-jones    bobanJo
ahbobanita     BOBAN               Bobanson           ahboban
```

The final pattern should be: `/\v%V\c\a@<!boban\a@!`

There should be 3 matches:

- "boban"
- "BOBAN"
- the "boban" from "boban-jones".

## Exercise 3 - Singaporean Boban's

Search for Singaporean ahboban's, but not ahbobanita's.

We'll need a positive look behind for "ah" and a negative look ahead for "ita".

According to Professor Zij, prefixing a name with "ah" is a "diminutive" form.
Equivalent to "Fred" becoming "Freddy" in English.
It's used in Singapore and some other nearby Asian areas like Hong Kong.

- highlight the below block with linewise highlight then hit `<escape>`
- start a magic case insensitive search on the block with `/\v%V\c`
- add `boban` (all the terms should light up)
- add `(ita)@!` (meaning "must not have "ita" following it)
    - this should knock out "ahbobanita" and "bobanita"
- move your cursor back to just before the 'b' of "boban"
- add `(ah)@<=` (meaning "must have "ah" before it)

```
boban          bobanita            mcboban            boban-jones    bobanJo
ahbobanita     BOBAN               Bobanson           ahboban
```

The final pattern should be: `/\v%V\c(ah)@<=boban(ita)@!`

This should match just the boban in "ahboban".

# Recap: General syntax

## Very magic mode

The general syntax for look arounds in very magic mode is:

```
[ATOM]@[OPERATOR]

where OPERATOR is one of:

=   positive lookahead                   :help /\@=
!   negative lookahead  (first example)  :help /\@!
<=  positive lookbehind (second example) :help /\@<=
<!  negative lookbehind                  :help /\@<!
```

Here "atom" just means a little atomic chunk of pattern.
If it's complex pattern like "ita" in the previous example, wrap it in brackets to make it an atom.
If it's already something simple like `\d` then it can have a look around attached to it without brackets.

## Magic mode

Above `@` is not escaped because we're assuming very magic mode.

In magic mode and below, use `\@`.

e.g. in magic mode `\d\@<!` means "not preceded by a digit"

## How to remember this

Overall think of `@` as the "look around" operator and the more general form as

```
[ATOM]@[DIRECTION][PARITY] where:

DIRECTION: < for backwards and not specified for forwards
PARITY:    = for "positive" and ! for "negative"
```

## Other operators

There are other operators that can be used with `@` but we'll just focus on these 4 today.

You can see the full list by doing `:help pattern` then searching `^\\@` (all lines starting with "\@").

# Conclusion

Look around logic is useful, particularly for find-replace scenarios and syntax highlighting.

It's a concept that goes beyond vim so it's worth having some basic awareness of it.

Vim's syntax is unusual and don't feel bad if you can't remember it - the main thing is you know how to find
it for those few times you need it.
