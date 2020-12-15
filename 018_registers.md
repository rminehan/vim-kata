# Registers

Built into vim is a predefined set of "registers" or variables.

Each letter of the alphabet has one (and there's a few other miscellaneous ones).

It would be as if you had scala code like:

```scala
var a = ""
var b = ""
var c = ""
...
var z = ""
```

Today we'll focus on using alphabetical registers as "extra clipboards".

# Many registers

Registers are a place you can store bits of text that you want to use later.

Think of it like having 26 clipboards. 

One of the annoying things about having one OS clipboard is that you only have only one buffer to save text into,
and each new copy writes over your previous one (unless you have a clipboard manager (which you should!))

# Vimscript

Registers are special vimscript variables (marked by the `@` character).

For example to set register `a` to the value "Boban" we can do: `let @a = 'Boban'<enter>`.

If you want to see the value of a register `a` you can do: `echo @a`

To copy from register `a` into `b`: `let @b = @a`

# Using registers whilst editing

The `"` character is used for copying and pasting between registers and the buffer.

You can paste the contents of a register into your buffer by doing:

```
# Paste after the cursor
"[REGISTER LETTER]p

# e.g. paste from register a after the cursor:
"ap

# Paste on the cursor
"[REGISTER LETTER]P

# e.g. paste from register a onto the cursor:
"aP
```

You can also copy text from your buffer into a register by prefixing a yank command with a register:

```
"[REGISTER LETTER]y[TEXT OBJECT]

# e.g. yank the current paragraph into register a:
"ayip
```

This is the same yank command we're familiar with - we've just added a register in front of it.

You can also delete text into a register:

```
"[REGISTER LETTER]d[TEXT OBJECT]

# e.g. delete the current little word into register a:
"adiw
```

The same applies to the change operator `c`.

So for these operators (`ydc`), this is actually a more general form of the "operator + text object" we've been using.

A natural question might be: "When I don't explicitly use a register, where is the text going?"
We'll get to that another time.

# Exercises

## Exercise 1

At the time of writing the paragraph below, we didn't know the names we'd use, so we just put in place holders "a", "b", "c" etc...

The names we decided on are:

- a - Thilo the Frankfurter
- b - Zij the linguistifier
- c - Lulu the Amongerer of Us'es

Load these into registers then use your register pasting skills to fill in the names below.

- load Thilo into register `a`: `:let @a = 'Thilo the Frankfurter'<enter>`
- load Zij into register `b`:   `:let @b = 'Zij the linguistifier'<enter>`
- load Lulu into register `c`:  `:let @c = "Lulu the Amongerer of Us'es"<enter>` (note the double quotes here)
- move your cursor to the first "a"
- delete it with `diW`
- paste in register `a` by doing: `"aP`
- move your cursor to the first "b"
- delete it with `diW`
- paste in register `b` by doing: `"bP`
- continue on until all are substituted

```
There was a man called "a" - a friend of the people, particularly those who like eating frankfurts.
Across the river lived "b" and "c", they were thick as thieves.
But today "a" and "b" thought "c" was sus.
```

Note that here a find replace or some dot operator foo might have been better. We just did it this way for teaching purposes.

## Exercise 2

Vimscript uses the `.` operator for string concatenation (:hmmm-parrot:)

Use this newfound knowledge to populate the `d` register with:

"Thilo the Frankfurter, Zij the linguistifier, Lulu the Amongerer of Us'es"

- this is registers `a`, `b` and `c` from exercise 1 all concatenated with commas between.
- do `let @d = @a . ', ' . @b . ', ' . @c`
- check it with `echo @d`

## Exercise 3

The descriptions below need to be matched to the correct developers.

Use registers to delete them all and then paste them back into the right places:

- (the first one looks like it's for Paul - we'll put it in register `p`)
- move your cursor somewhere inside the first description
- do `"pdi"` which is:
    - `"p` - register p
    - `d` - delete
    - `i"` - inside double quotes (text object)
- check it's in register `p` by doing `echo @p`
- (the next looks like it's for Thilo - we'll put it in register `t`)
- move your cursor somewhere in the next description
- do `"tdi"`
    - this is the same as Paul's one, but we're deleting into register `t`
- check it's in register `t` by doing `echo @t`
- (the last one of course describes Enxhell, the lack of respect is the give away)
- put your cursor inside the description and do `"edi"`
- (now they're all deleted and in registers, we paste them back in)
- move your cursor to the left double quote for Enxhell
- do `"ep` (which is "paste from register `e` after the cursor")
- move your cursor to the left double quote for Paul
- do `"pp` (which is "paste from register `p` after the cursor")
- (for the last one we'll practice pasting on the cursor)
- move your cursor to the right double quote for Thilo
- do `"tP` (which is "paste from register `t` on the cursor")

```
enxhell: "plans to name his firstborn 'ZIO'"
paul:    "sassy German with a healthy fear of production"
thilo:   "a no-nonsense youngster who needs to respect his elders"
```

## Exercise 4 (optional bonus)

When you are pushing data into a register (using delete, yank, change etc...),

if you use the capital letter for the register, it's understood to mean "append",

rather than writing over it.

Use this newfound knowledge to make Jonathan the winner of the 2020 Sassy Parrot Award in register `w`.

- do `let @w = 'And the winner of the 2020 Sassy Parrot award is: '<enter>`
- put your cursor on a Jonathan word (e.g. that one just to the left)
- do `"Wyiw` which is:
    - `"W` - register `w` (append mode)
    - `y` - yank
    - `iw` - inside little word
    - ie. append the little word under the cursor to register `w`
- do `echo @w` to check it worked

# Summary

There's 26 registers we can use for storing bits of text.

You can use vimscript to work with these registers programmatically.

Likewise you can use common operators like `d` and `y` to push text into them from your buffer,
and `p` or `P` to paste from them into the buffer.
