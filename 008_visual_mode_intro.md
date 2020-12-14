# Visual Mode

Visual mode is analogous to highlighting text in other editors. It is a separate editing mode though.

Because developers are used to using highlighting from other editors, they can sometimes use visual mode as a crutch,
when vim has superior ways to manipulate text (in particular the operator+text object form).
(That's why I've introduced it relatively late, to hopefully reinforce better behaviors first)

For today we're going to focus on using visual mode for building ad-hoc text objects,
where vim doesn't have a nice built-in one.

# Visual mode shortcuts

To get started, we'll cover a subset of visual mode today.

Use `v` to go into visual mode.

From there use your motions to extend the visually selected area, e.g.

- `l` to go right one character
- `h` to go left one character 
- `w` to go right one word
- `b` to go back one word

Use `o` to switch which anchor your cursor is controlling.

Use `<escape>` or `v` to return to normal mode.

Use `gv` from normal to reselect your last

# Changing things

From previous kata, we're used to the "operator + text object" form for manipulating text.

When you're in visual mode, you've already implicitly defined the text object as the highlighted text.

Hence you just need to type an operator (e.g. `gU` for upper case) and it will get applied to that text.

For example, highlight text then press:

- `d` to delete it and return to normal mode
- `c` to delete it and go to insert mode
- `gU` to upper case it

# Exercises

## Exercise 1 - deleting stuff

Delete the chunk of upper case text in the block below:

- put your cursor on the 'U' in "UPPER_CASE"
- do `v` to enter visual mode
- press `e` repeatedly until the visual area has covered "WIN"
- press `d` to delete the chunk

```
lowercase snake_case dash-case UPPER_CASE FOR THE WIN all done
```

(note in this case we _could_ have used `4daw` or `daw...` and they would have also deleted that extra space)

## Exercise 2 - upper casing stuff

Upper case all the sql keywords in the query below: (that is everything except "leads" and "timestamp")

- put your cursor on the 's' in "select"
- do `v` to enter visual mode
- press `j` to extend the selection down (it should now cover "select *" and the 'f' in "from")
- press `e` to extend the selection to the end of "from"
- do `gU` to uppercase the selection
- move your cursor to the 'o' in "order"
- do `v` to enter visual mode
- press `ee` to extend the selection across "order by"
- do `gU` to uppercase the selection
- move your cursor to the 'd' in "desc"
- do `v` to enter visual mode
- press `j` to extend the selection down a line (it should encompass the whole last line)
- do `gU` to uppercase the selection

```sql
select *
from leads
order by 'timestamp' desc
limit 30
```

## Exercise 3 - changing stuff

Replace the text inside the quotes with "Boban":

- move your cursor to the 'E' in "Enxhell"
- do `v` to enter visual mode
- press `e` a few times until the word "machines" is highlighted (but not the double quote)
- do `c` (this will delete the text and put you in insert mode)
- do `Boban<escape>`

```
And our favourite Albanian is: "Enxhell the trainer of machines"
```

## Exercise 4 - using text objects

You can use existing text objects like `ip` or `iW` to extend a text object.

Create a visual selection that spans the two paragraphs below and the final line:

- move your cursor to anywhere in the first paragraph
- do `v` to enter visual mode
- do `ip` to extend the visual selection over the whole paragraph  (your cursor should be at the end of the first paragraph)
- press `j` (down) until your cursor is in the next paragraph
- do `ip` to further extend the visual selection to include the second paragraph
- press escape to return to normal mode
- do `gv` to reselect your previous selection
- do `o` to move your cursor to the top anchor
- press `k` (up) to move the selection up one line to include the empty line
- do `o` to move your cursor to the bottom anchor
- press `j` (down) to move the selection down one line to include the empty line
- do `d` to delete the whole area (this should delete the entire example)

```

Abc
Def
Ghi


012
345
678

```

## (Bonus) exercise 5 - replacing characters

When in visual mode, you can use `r` (replace) to replace every character highlighted with a new one.

Replace he who shouldn't be named with just 'x' characters in the snippet below:

- move your cursor to anywhere in the first "Voldemort"
- do `viw` to highlight the word
- do `rx` to replace the word with x's
- move your cursor to anywhere in the next "Voldemort" and repeat above

```
And then I was like "Voldemort"? Who the bloomin' heck is "Voldemort"? Blimey...
```

# Summary

Visual mode is good for creating ad-hoc text objects to use with operations.

If there already an existing text object for your needs, try to use it instead though.
That will make you a stronger vimmer over time.
