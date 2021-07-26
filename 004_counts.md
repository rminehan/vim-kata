# Counts

You can use counts to perform multiple changes in one go.

A count means typing a number before typing your typical normal command.

For example, to delete 3 words forward, you can do `3dw`. Breaking that down:

- `3` is the count
- `dw` is a regular "operation + text object" command
    - `d` delete
    - `w` forward one word (here a motion is being converted to a text object)

The dot operator will include the count as part of the change. So pressing `.` would cause another 3 words to be deleted.

# Exercise

## Exercise 1 - deleting

Every mention of Boban in the text below is followed by a flowery title containing 2 words.

Remove all those titles.

- put your cursor on the "t" of "the magnificent" on the first line
- do `2dw` (this should cause "the magnificent" to disappear)
- move your cursor to the "t" on "the mighty" on the next line
- do either `2dw` or `.` to delete "the mighty"
- carry on deleting the remaining titles in the same way 

```
Boban the magnificent said to his subjects:

I am Boban the mighty, and I fought many battles as Boban the brave.

Spread the word about the great deeds of Boban the humble.
```

## Exercise 2 - deleting++

You might have noticed that our approach for exercise 1 wasn't that clean.

It left some spaces between "Boban" and punctuation marks that came after the title (e.g. "," and ".").

It also required us to start our cursor on the "t" in each "the"...
(we used to luxuriating in being able to put our cursor anywhere in the text object)

What we want is a text object that will delete the little word and also any surrounding whitespace:
Introducing `aw`, or "around little word".

So we have sibling text objects:

- `iw` - inside little word
- `aw` - around little word

Let's try again on the text above.

- press `u` repeatedly to undo the changes we made in exercise 1
(or `git checkout` local changes to the file, or use `:e!` to reload the file from disk if all your changes are unsaved)
- put your cursor anywhere on the "the" in "the magnificent"
- do `2daw`, ie. "do `daw` two times where `daw` is delete around word"
- move your cursor to anywhere in the "the" for the other titles and either to `2daw` or `.` for each

## Exercise 3 - count + motion

Counts can be combined with motions.

For example `j` means to go down 1 line and `30j` means to go down 30 lines.

We can use motions as text objects, hence `d30j` would mean "delete from here down 30 lines (inclusive)".
Because it's inclusive it deletes 31 lines overall.

Use this new foo to delete the lines that start with "DELETE ME":

- move your cursor to anywhere in the first line that starts with "DELETE ME"
- do `d2j` (ie. delete from here down to 2 lines below)
- move your cursor to the next "DELETE ME"
- do `d3j` (ie. delete from here down to 3 lines below)

```
Don't delete me!
Keep me too!
DELETE ME now!
DELETE ME do it nowwww!
DELETE ME forever

Keep this line too.
DELETE ME or else
DELETE ME to stay jiving
DELETE ME before I learn to feel pain
DELETE ME quick!
```

This is why Rohan uses those crazy relative line numbers. It makes it very fast to delete big chunks of code.

# Counts with motions with counts?

You could do something like: `3d4w` which means "do 3 times: delete forward 4 words".

This is the same as just deleting forward 12 words. So you could simplify that to:

- `12dw`: "do 12 times: delete forward a word"
- `d12w`: "delete forward 12 words"

# Counts with `c` (change)?

Running `3ciw` then entering new text like "Thilo" will not change the next 3 words into 3 "Thilo"s.

It will instead delete the 3 words, go into insert mode and enter "Thilo". We'll have to find those extra Thilo's somewhere else.

# Counts with `y` (yank)?

Doesn't make any sense to copy the same text repeatedly. :idempotent-parrot:

# Counts vs "Interactive"

In a case like deleting words or indenting text, some people prefer to just do the regular action (e.g. `dw`),
then interactively use `.` to duplicate the change, rather than trying to count beforehand how many times to do it. 

This "interactive" approach can be more intuitive and the greater level of granularity makes it easier to undo
if you go too far applying your action.

For example if you do `6dw` and overshoot by one word, pressing `u` will restore all 6 words and then you'd do `5dw`.
If instead you just did `dw` and zealously hammered `.`, if you overshot you can just press `u` to undo what you overshot.
