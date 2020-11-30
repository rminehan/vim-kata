# Motions as text objects

In [kata 0](./000_operations_and_text_objects.md) we looked at how you can combine operations with text objects.

Vim also has a concept of "motions" (navigational movements) which can be converted into text objects.

Some examples of motions are:

- `w` - ("word") moves you to the next little word
- `e` - ("end") moves you to the end of the current little word
- `b` - ("back") moves you back to the previous little word

The above have analogous "big" uppercase motions, e.g. `W` moves you to the start of the next _big_ word.

When you enter a operator (e.g. `d` for delete), you can use most motions in place of a text object.
Vim will create a text object from your current position to the position the motion would take you to.

For example if you do `dw` then you are deleting forward a word.

# Exercises

## Exercise 1

Remove the "Foo" suffixes from the words below.

For each word with a "Foo":

- move your cursor to the "F"
- do `de` (delete to end) (or you could use `.` after the first deletion)

```
We wantFoo to learnFoo some awesome vimFoo.
```

## Exercise 2

The Boban's have invaded this block of text. Remove the "Boban" prefix from all infected words.

For each word with a "Boban" prefix:

- move your cursor to the character after "Boban" (e.g. for "BobanZij" put your cursor on "Z")
- do `db` (delete back) (or use `.` for subsequent deletions)

```
BobanZij said to BobanLulu that BobanThilo is a real Bobangreat guy.
```

## Exercise 3

When writing out this letter, our intern forgot to add "His-Highness-the-Great" before some of the "Boban"s.

Fix this by copying the text His-Highness-the-Great from the first line and pasting it before each subsequent "Boban".

- move your cursor to the first "H" in "His-Highness-the-Great
- do `yW` (yank big word)
- (this will copy "His-Highness-the-Great " - note the trailing space)
- move your cursor to the leading "B" of "Boban" on the next line
- do `P` to paste at that position (it's important to use upper case P here)
- repeat moving the cursor and pasting for the other Bobans (note you can use `.` instead of `P` for subsequent pastes)

```
A letter regarding His-Highness-the-Great Boban to the people.

Your great King Boban would like you to all to know,

that as King Boban he wishes you all.

This letter is officially sanctioned propaganda from Boban
```

## Exercise 4

In the text below, each line got duplicated.

We want to delete the 2nd and 3rd line:

- move your cursor to somewhere on the second line
- do `dj` (`d` delete + `j` down motion)

```
Hello there,
Hello there,
this is Boban
this is Boban
```

# Comparing motions with text objects with words

For wordy operations, it's often better practice to use vanilla text objects instead of motion based text objects.

This is because they will work regardless of what position your cursor is at.

For example suppose you want to uppercase all the "Boban"s from this sentence:

```
boban is what Boban does.
```

The best way is to use `gUiw` (ie. `gU` upper case + `iw` in word) because that will work from anywhere in the word.

To use `gUw` (ie. `gU` upper case + `w` forward a word) will only work if your cursor is at the front of the word.

Another reason `gUiw` is better is because your change will be easier to apply to other words you want to uppercase,
in that you can be anywhere in that word and just press `.`.

So even though vanilla text objects often have an extra `i` or `a` character, they are a better way to go,
and using motions will often an extra keypress anyway to move your cursor to the right location.

(For line based operations, often there isn't a suitable existing text object and motions make more sense)

# Terminology

I've been using the terminology of "text objects" as the more general concept of a block of text,
and motions as a type of movement which can be used to create a text object.

Some vimmer's (and the vim docs) will use "motion" to mean what I refer to as a "text object".

So I've gone rogue here. :rogue-pirate-rohan-parrot:
