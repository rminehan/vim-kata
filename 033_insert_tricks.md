# Insert tricks

Today is a miscellaneous grab-bag of tricks related to insert mode.

# Exercises

## Exercise 1

Insert mode has a few control based shortcuts for modifying text without leaving insert mode:

- `<c-w>` - delete back a word
- `<c-u>` - delete back a chunk
- `<c-r>[REGISTER]` - insert from register

The first two are actually bash/emacs style shortcuts and they also work in command mode,
so there's value in learning them beyond just insert mode.

The last one we covered in [kata 20](020_misc_registers.md).

Shortcuts like `<c-w>` and `<c-u>` are also sometimes preferable to returning to normal mode
because it keeps your change atomic, making it more repeatable, and keeping your undo history cleaner.

For example if you'd typed: "His name was Bban" then returned to normal mode to jump back and
fixed "Bban", then your last change will just be that fix and won't include the entire sentence.

We'll play with these shortcuts:

- put your cursor on the first line of backticks and do `o`
- type `His name was Bobn` (note the typo)
- instead of returning to normal mode, do `<c-w>` and retype `Boban<escape>`
- hit `.` to repeat the line insert
- use `A` to continue inserting at the end of the line and add `ita` for Bobanita
- do `<c-u>` - this will delete the "ita" just entered
- do `<c-w>` then type `Enxhell<escape>`
- do `k` to go up a line then `.` to repeat the change
- do `A` to continue editing and type `, old and wise`
- do `<c-u>` to wipe out the old and wise
- do `<c-u>` again to wipe to the start of the line
- do `<c-u>` again to wipe the line completely
- then `<escape>` to return to normal mode

```
```

## Exercise 2

In [kata 28](028_built_in_marks.md) we looked at mark `^` which automatically marks the position of the last
insertion made in the buffer.

Often the reason you want to jump back to that point is to continue inserting text.
For these cases `gi` is much better - it jumps you back to the end of your last insert and
puts you straight into insert mode. 

We'll reuse kata 28 exercise 1 to practice `gi`:

- move your cursor to the closing bracket on `val boban = person()`
- (we want to call the person function)
- do `i` to start typing
- enter some random id like "2304234" and a comma and a space
- (oh no! We're old and senile and we forgot what the next argument is, is it name or age?)
- return to normal mode, move your cursor to `person` and hit `*`
- (this represents us searching for the definition of the function)
- (okay - we can see it's age _then_ name)
- do `gi` to jump back to insert mode at your last change and enter `29, "Boban"<escape>`

```scala
def person(id: String, age: Int, firstName: String): Person = ...

val boban = person()
```

## Exercise 3

Each time you make an insert, the position of that insert gets added to a jump list in the buffer.

You can jump back and forth between those using `g;` and `g,`.

Note that `;` and `,` are also the forward and backward keys for linewise character matches with `f` and `t`
from [kata 15](015_linewise_jumps.md).

- go to the "Boban" line and do `Aita<escape>` to append "ita"
- go to the "Enxhel" line and do `Al<escape>` to fix the typo
- do `jjj` to move your cursor off the line of your last insert
- do `g;` a few times to jump back through the insert from this exercise and previous exercises
- use `g,` to come back to this one
- use `gi` to continue editing the Enxhell line and add `, trainer of pythons who is <c-r>=8*2<enter> years old`

```
Boban

Enxhel
```

# Insert jump lists are specific to buffers

Keep in mind that each buffer has its own insert jump list.

So `gi` will take you to your most recent inser in the current buffer,
but that might not be the most recent inser you've made overall.

# What about the regular jump list

There is a more general jump list which you can navigate using `<c-o>` and `<c-i>`.

Overall I don't find the jump list that useful as vim isn't very consistent or intuitive
about what constitutes a "jump".

For example:

- going up and down paragraphs using `{`/`}` counts as jumping (which I think pollutes the jump list)
- going up and down a fixed number of lines (e.g. `50k` for "up 50 lines") _doesn't_ count as a jump

Usually I just want to get back to the last point where I was editing and find that using
the insert jump list or automatic marks like `^` and `.` work well.

For switching quickly between buffers I have backspace bound to the alternate buffer which also removes
a lot of use cases for the jump list.

# Conclusion

Whilst normal mode is the norm, there's still value in learning these little insert mode tricks.

They can make editing a bit less clumsy when you want to make small fixes from insert mode,
and they'll keep your undo history and insert jump list cleaner.

Shortcuts like `<c-u>` and `<c-w>` exist in a similar form outside vim (e.g. bash)
so there's extra value in learning them.

Finally the insert jump list and the related shortcuts are a nice simple way to jump back through
inserts from the current buffer which often act as natural bookmarks for the places you've been working on.
