# Advanced regex 8 - groups and back references

All the advanced regex kata so far have focused on how to find matches,
and when we have used the `substitute` command, the substitution has always been a simple string.

Today's kata is looking at more sophisticated tricks for matching and replacing text using its internal structure.

# Back references

All our `substitute` examples so far have had simple substitution logic.
Usually whatever we match just gets replaced with some fixed text.

Suppose though we were wanting to swap names in the form "[LAST NAME]-[FIRST NAME]" to "[FIRST NAME]-[LAST NAME]",

e.g. "Jones-Boban" becomes "Boban-Jones".

This involves identifying subsections of the match and using those in the replacement logic.

If you highlight the text below and run this substitution you should get the desired result.
You can run it again to reverse the effect as it's self-inverting.

```vim
'<,'>s/\v(\a*)-(\a*)/\2-\1/g
```

```
Jones-Boban              TheGerman-Thilo
Luzsomething-Enxhell     Bobanita-Hayworth
```

Let's break this down:

```
'<,'>  s  /\v(\a*)-(\a*)    /\2-\1/   g           FIND
           -------------     ------               '<,'>   line range corresponding to the visual selection
               FIND         REPLACE               \v      very magic (saves us having to escape the brackets)
                                                  (\a*)   0 or more English letters forming capture group 1
    REPLACE                                       -       literal hyphen
    \2   back reference to capture group 2        (\a*)   0 or more English letters forming capture group 2
    -    literal hyphen
    \1   back reference to capture group 1
```

When we put brackets around terms in the search pattern, they become match groups
and get numbered 1,2,3 etc... from left to right based on their position in the search pattern.

Those numbers can be used in "back references" to refer to that matched text later on.

The replacement string here is formed by using back references to the match.

# Exercises

See the [setup guide](advanced_regex_exercises_setup.md).

## Exercise 1 - simple palindromes

Find the length 4 or 5 digit palindromes in the area below.

(A palindrome is a string that's the same forwards or backwards, e.g "0123210")

To achieve this, we can put back references into the search itself (see appendix).

We'll start by just trying to match length 4 palindromes, then tweak our regex to match length 5 ones too.

- prepare the search area as usual
- do `/\v%V`
- put `(\d)(\d)` to match 2 digits
    - each digit has its own capture groups: 1 and 2
- after the above add `\2\1`
    - now it should match length 4 palindromes only
- after the second `(\d)`, add in `\d?` meaning "optional digit"
    - now it should match all the length 4 and 5 palindromes
- wrap word boundaries `<>` around the core of the match (to stop us matching palindromes inside longer strings)

```
  SHOULD MATCH           SHOULDN'T MATCH
     0110                      0
     0000                     121
    12921                     1222
    99899                    876 78
     1221                     12320
                             123321
                            12344321
                               33
                             abba
                             abcba
```

The final pattern is `\v%V<(\d)(\d)\d?\2\1>`.

It makes sure that the first and last digit matched are the same by putting `\1` at the end.

It makes sure that the second and second last digit matched are the same by putting `\2` second from the end.

It allows for an optional middle digit.
When that matches something you have a length 5 palindrome, otherwise you have a length 4 palindrome.

Matching palindromes of any length can be achieved by regex engines that support recursion
(which technically is no longer "regular" expressions in the strict sense).
Vim's regex engine doesn't support recursion but there are other engines that do, like perl's.

## Exercise 2 - capitalization

Change each duplicate name below to have a capitalized first letter.

We'll define a name as something with at least 4 letters and word boundaries.
That should prevent words like "the" and "34592" being considered names.

First we'll just search the text below for names that occur twice using a back reference.
Later we'll do replacement.

- prepare the search area as usual
- do `\v%V`
- add `(\a{4,})` (this captures 4+ letter words into group 1)
- put word boundaries (`<` and `>`) around the above
- add `.{-}` to the end of the match
    - this lazily matches to the end of the line currently
- add `\1` (this looks for repetitions of the earlier name)
    - so far it will only match Marianna's sentence as the case is the same (due to `%V` disabling case insensitivity)
- put `\c` after `%V`
- put word boundaries around `\1` to remove some imposter matches
- hit enter to lock that in

```
   CREWMATES                        IMPOSTERS
   boban the boban                  enxhell the enxhellic
   the bobanita and the bobanita    R2D2 and the other one r2d2
   Lola and her friend lola         bola and her friend Ebola
   Marianna who is Marianna         Jo the jo
                                    angelina7 of planet angelina
```

At this point your search should be `\v%V\c<(\a{4,})>.{-}<\1>`

Now we'll substitute.

We want to capitalize the second occurence of each name.
We'll put the zoom operators `\zs` and `\ze` to narrow the matched part to just the second name,
then use a spiffy trick `\u` to capitalize the first letter in the replace text.
See `:help sub-replace-special`.

- do `:'<,'>s/` to start a substitute on the last visual selection (you'll have to type in the range manually)
- do `<c-r>/` to insert in the last search term
- put `\zs` and `\ze` around `\1` to limit the match to just it
- append `/` to begin replacing
- do `\u\1` which means "group 1 with the first character uppercased"
- add `/g` in case there's multiple matches per line
- lock it in with enter

The transformation should have been:

```
   boban the boban                               boban the Boban
   the bobanita and the Bobanita        ---->    the bobanita and the Bobanita
   Lola and her friend lola                      Lola and her friend Lola
   Marianna who is Marianna                      Marianna who is Marianna
```

Optionally hit `u` to undo and try other case variants in place of `\u` like:

- `\l` to lowercase the next character (e.g. "Marianna who is marianna")
- `\U` to uppercase all subsequent characters until hitting an `\E`
- `\L` to lowercase all subsequent characters until hitting an `\E`

Note that `\u`, `\l` etc... have this meaning when they're in the _replacement_ section.
In the match section, `\u` actually means an "upper case letter" and `\U` means a "not upper case letter".
There's an example of this in the appendix.
See `:help /\u` or `:help /\U` for the search versions and `:help s/\u` or `:help s/\U` for the replacement versions.

## Exercise 3

For this exercise we want to completely uppercase all "Bobany" words where a Bobany word is defined as one of:

- Bob
- Boba
- Boban
- Bobany

So:

- "Boban" -> "BOBAN"
- "Bobany" -> "BOBANY"
- "Bo" wouldn't change
- "Bobany2" wouldn't change

Before we dive into the exercise, we need to think about a smart way to match these words.

The essence of Bobany-ness is:

> It must be a substring of "Bobany" from position 0, but it needs to be at least length 3

The length 3 condition rules out "", "B", and "Bo".

To match this we _could_ do something like `(Bob|Boba|Boban|Bobany)`.
The duplication isn't nice though and it would be easy for a typo to sneak in.

We could at least extract out the "Bob" prefix: `Bob(|a|an|any)`,
ie. match "Bob" and then either: the empty string, "a", "an" or "any".

Be careful of order too. A `(..|..|..)` will short circuit out on the first match it gets.
So if the text were "Boban", it would just match the "Bob" because the empty string case would win out.
So we'd need to reverse the order - longest to shortest: `Bob(any|an|a|)`

This will work, but as a general strategy it's not very good particularly if we had a really long string.

Thankfully vim has a thing for this called a "sequence" (`:help /\%[`).
You give it a sequence of characters and it will match as far as it can into that sequence.

The syntax in very magic mode is `%[...]` and in magic mode down it's `\%[...]`.

For example `%[abc]` will match "a", "ab" and "abc".

Let's first use this to just match out Bobany words:

- prepare the search as usual
- do `/\v%V`
- add `bob` to require matching at least 3 letters
- add a `\c` after `%V` to make it case insensitive
- add a sequence `%[any]` after `bob`
- put word boundaries around `bob%[any]` to avoid partially matching inside words like "Bobby"
- lock in the search with enter

```
  CREWMATES         IMPOSTERS
  Boban             Bobby
  bob               Bo
  boBa              Yaboban
  bobanY            B
```

The final pattern should be: `\v%V\c<bob%[any]>`

Now to substitute.

The task is to completely uppercase the matches,
which we can do by putting with `\U` before a back reference in our replacement (see `:help s/\U`).

- do `:'<,'>s/` to start a substitution
- do `<c-r>/` to insert what's in the `/` register (the last search)
- do `/` to start replacing
- put in `\0` (referring to the entire match)
- put `\U` just before it (to uppercase it)
- add `/g` at the end to allow multiple replacements per line (even though none of the imposters should match)
- hit enter

The final substitution should be: `'<,'>s/\v%V\c<bob%[any]>/\U\0/g`

All the crewmates above should have gotten completely uppercased and none of the imposters.

For extra points, hit `u` to undo and run the substitution again using `\U&` instead of `\U\0`.
The appendix explains that `\0` and `&` represents the entire match.

Hopefully you can see how sequences would be useful in some situations.
Vim itself is very "sequency" in how ex commands can be entered.

For example all of the following ex commands quit: `q`, `qu`, `qui`, `quit`.
You could imagine vim using a regex like `\vq%[uit]` to check them.
Note `q` isn't in the sequence, otherwise the empty string would match.

Another example is `:only` to close all splits but the current.
You can do `on`, `onl` and `only` which is like `\von%[ly]`.

# Summary

Groups and back references open up the possibility of more powerful searches
where there is an internal relationship between:

- sections of text in your matches
- the match and the replacement

Vim also has powerful special casing tricks you can do with replacement strings (`:help sub-replace-expression`).

As an extra bonus we learnt about sequences which are good for substring based matching.

# Appendix

## Magicness

In magic mode and down, brackets need to be escaped.
The equivalent magic version of our opening example would be:

```vim
'<,'>s/\(\a*\)-\(\a*\)/\2-\1/
```

## Look arounds

Note that brackets associated with look arounds also get included in the numbering.

### Negative example

This example looks for digit patterns where the second pair of digits isn't "00".

```vim
'<,'>s/\v(\d\d)-(00)@!(\d\d)/1:'\1', 2:'\2', 3:'\3'
"        ------ ----  ------ ----------------------
"           1     2      3     label each group
"                               and print them
```

The first line matches and gets replaced.
You can see from the output that the lookahead (group 2) is empty.
This makes sense as the whole point of it is to _not_ match anything.

```
12-34
99-00

replaces to:

1:'12', 2:'', 3:'34'
99-00                   <--- doesn't match because of the negative lookahead
```

### Positive example

Switching the lookahead to a positive one:

```vim
'<,'>s/\v(\d\d)-(00)@=(\d\d)/1:'\1', 2:'\2', 3:'\3'
"                    ^ changed
```

This time the second line matches and gets replaced.
The lookahead (group 2) contains "00".

```
12-34
99-00

replaces to:

12-34                    <-- doesn't match because of the positive lookahead
1:'99', 2:'00', 3:'00'
```

## Nested groups

You can have groups within groups.

They're still numbered based on the order that opening brackets are encountered moving left to right through the pattern.

For example

```vim
'<,'>s/\v((\d\d)-(\d\d))--((\d\d)-(\d\d))/1:\1\r4:\4\r2:\2\r3:\3\r5:\5\r6:\6
"        ---------------  --------------- ----------------------------------
"               1                 4          label and print every group
"         ------ ------    ------ ------          on its own line 
"           2       3         5      6          (\r means new line)
```

```
22-33--55-66

replaces to:

1:22-33
4:55-66
2:22
3:33
5:55
6:66
```

## The zero back reference

`\0` is a back reference to everything that was matched. An alternative is `&`.

Just think of it as if your entire pattern was surrounded by another set of brackets.
Because those brackets open before any others, it's group 0.

Following the example above:

```vim
'<,'>s/\v((\d\d)-(\d\d))--((\d\d)-(\d\d))/0:\0\r\&:&
"                                         ----------
"                                          Replace
```

```
22-33--55-66

replaces to:

0:22-33--55-66
&:22-33--55-66
```

Note above to make the literal `&` appear, I had to escape it.

## Non-capturing groups

If you put `%` before a `(...)` group, it doesn't capture it.

For example: apply this below:

```
s/\v(\w+)-%(\d+)-(\w+)/\1-\2

# Before
abc-0123-def

# After
abc-def
```

Above, `\2` refers to the third bracketed group, because the middle digit one wasn't counted.

`:help /\%(` also mentions:

> This allows using more groups and it's a little bit faster.

Note in magic mode and down you'd use `\%`.

## Size limit

Vim only supports up to the 9 groups in a pattern.

Simply running a search like this:

```vim
/\v(.)(.)(.)(.)(.)(.)(.)(.)(.)(.)
"   1  2  3  4  5  6  7  8  9  10
```

will fail with an error like:

```
E872: (NFA regexp) Too many '('
```

This means that back references only go up to the letter 9.

If we mark one of them as non-capturing though, we just fit under the limit:

```vim
" This will find blocks of 9 characters
/\v(.)(.)(.)(.)(.)(.)(.)(.)(.)%(.)
"   1  2  3  4  5  6  7  8  9 ^ non capturing
```

# Using back references in searches

Back references can also be used in searches (not just the replace part).

This is very useful when you want to relate one part of a match to another.
For example we used heavily in the exercises for palindromes and matching duplicate words on a line.

Suppose we want to find duplicated names like "BobanBoban" and replace them with just their single name.
So far none of the advanced regex tools we've seen would be able to do this well.

A pattern like `/\v(\a+)\1` means: "one or more letters followed by those same characters just matched"

Below it matches "BobanBoban", "BobanitaBobanita", "ee" (from "seen") and "EnxhellEnxhell".

```
BobanBoban said to BobanitaBobanita,
have you seen cousin EnxhellEnxhell?
```

To stop it matching things like "seen" we'll require that the name has:

- a leading capital letter
- at least 2 letters
- word boundaries on either side

This would be: `/\v<(\u\a+)\1>` (here `\u` means "upper case English letter" - see `:help /\u`)

We can use `'<,'>s/\v<(\u\a+)\1>/\1/g` to replace the matches with just `\1` which has the effect of removing
the duplicate.

```
'<,'>s/\v<(\u\a+)\1>/\1/g

'<,'>  s  /\v<(\u\a+)\1>  /\1      /g
           -------------   --       -
           same pattern   replace   replace
                           with     multiple matches
                         group 1    per line
```

```
BobanBoban said to BobanitaBobanita,
have you seen cousin EnxhellEnxhell?
```

replaces to:

```
Boban said to Bobanita,
have you seen cousin Enxhell?
```
