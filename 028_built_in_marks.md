# Built in marks

If you remember from [kata 18 - registers](018_registers.md) and [kata 19 - special registers](019_special_registers.md),
you could define your own registers (a-z) and certain actions in the editor would populate special registers, e.g.

- yanking populated the `0` register
- searching populated the `/` register
- running ex commands populated the `:` register

There is a analogous concept for marks. Actions like editing and visually selecting automatically set marks in your editor.

Here's built in marks we'll cover today:

- `.` - the last change
- `^` - the last insertion
- `[` and `]` - the start and end of the last yanked text
- `<` and `>` - the start and end anchors for the last visual selection

Note that these are all buffer local.

# Exercises

## Exercise 1 - the last change

So often you want to jump to the last change you made in a buffer.

For example you might have been writing some code to call a function,
then halfway through calling it you wanted to check what the parameters are,
so you jump to the function, and perhaps go down some even deeper rabbit holes.
Eventually you know what arguments you want to pass and you're ready to start typing again.

In a situation like that the `.` and `^` marks are really useful.

- move your cursor to the closing bracket on `val boban = person()`
- (we want to call the person function)
- do `i` to start typing
- enter some random id like "2304234" and a comma and a space
- (oh no! We're old and senile and we forgot what the next argument is, is it name or age?)
- return to normal mode, move your cursor to `person` and hit `*`
- (this represents us searching for the definition of the function)
- (okay - we can see it's age _then_ name)
- do `'.` to return to the line with the last change
- (hmmm... that's put our cursor at the start of the line)
- do `<backtick>.` - that will send us right to the last character
- go into insert mode after the cursor with `a`
- add in an age (e.g. 20) and add a comma and a space
- (oops - another senior moment - what was that last argument again?)
- return to normal mode and hit `N` to jump back to the last search for "person"
- (okay it was name - just in case we forget after we jump back, put your cursor on it and yank it with `yiw`)
- do `<backtick>^` to return us to our last insertion
- go into insert mode after the cursor with `a` 
- enter "Boban"

```scala
def person(id: String, age: Int, name: String): Person = ...

val boban = person()
```

In this case `.` and `^` were marking the same position.

When are `.` and `^` different?
When your last change was not an insert, e.g. a paste, delete etc...
Or if your last insert didn't add any text, e.g. doing `i<escape>`.

Note also `gi` would have been even better here - we'll cover it in [kata 33](033_insert_tricks.md).

## Exercise 2

We were regaling our fellow developers with tales of how forgetful we are.

Then one of them said: Actually we want to rename "name" to "firstName".

You think to yourself and despite all your forgetfulness you remember that you happened to yank that word.

That yank would have been marked on the left and right by `[` and `]` respectively.

- to make it easier to see what's going on split the buffer with `:split` so that the instructions are in one buffer
- do `<backtick>[` to jump to the left of the last yanked area
- do `a` to just into insert after the cursor
- do `<backspace>firstN<escape>` to change it to first name
- quit the split with `:q`

(Note above if we had to make other replacements of "name" to "firstName" it would probably have been better
to just do `ciwfirstName` as that is easier to repeat with the mighty dot in other locations.

## Exercise 3

Then your friends say:

> Actually in the spirit of political correctness we've been instructed to make it half English half German.
> So we're calling it "firstNamen".

Right now it's "firstName" so we just need to jump to the back of that word and add an "n".

- split the buffer again with `:split`
- jump to the right hand mark for our last yank with `<backtick>]`
- (hmmmm - that mark has drifted because we modified the text since we yanked it)
- do `e` to move to the end of the word
- do `a` to go into insert after the cursor
- do `n<escape>` to add an "n"
- quit the split with `:q` 

# Ex commands on visual selections

Remember that a lot of ex commands take ranges (e.g. substitute). For example:

- `%` - entire buffer
- `4,6` - lines 4-6
- `.,+3` - the current line and the next 3 lines

In the last 2 cases the format for the range is `[START],[END]`

When you visually select something and then start an ex command with `:` vim will pre-populate the range for you
with this cryptic thing:

```
'<,'>
```

Now that we've learnt about marks we can make sense of this.

This is a range in the `[START],[END]` form with:

- start: `'<`
- end:   `'>`

A single quote here means "the line for this mark".

For example `'a,'b` would specify the range from mark `a` to mark `b`.

In our case

- `<` is the mark for the start anchor of the last visual selection (see intro)
- `>` is the mark for the end   anchor of the last visual selection

So in fact `'<,'>` means the: "the range of lines from the last visual selection".

So vim detects we had something highlighted and assumes (usually correctly) we wanted that
to be the range for our ex command, so it helpfully pre-populates that range for us.

If this _isn't_ what you want, you can clear it with `<c-u>`.
This is same `<c-u>` from bash (and maybe emacs?) for deleting to the start of the line.

# Conclusion

Learn to use these built in marks for quick navigation.

If it's too much mental strain to learn them all, just focus on trying to learn `.` (the last change in the buffer).
That one alone is probably more useful than the rest put together.
