# Misc registers

We're not finished with registers yet. There's still a few quirky ones to cover:

- `"` - the anonymous register
- `_` - the blackhole register
- `=` - the expression register

# A new shortcut

We're also introducing a new shortcut for inserting text from registers whilst you're in insert mode or command mode:

```
<c-r>[REGISTER NAME]
```

This can be a lot less fiddly than jumping back to normal mode, pasting from a register, then jumping back to insert mode.

# Exercises

# Exercise 1 - the anonymous register

When you perform operations like yank, delete, change and paste without explicitly mentioning a register,
the "anonymous" register is used.

It's character is `"` (the same as the character you use to dial up registers in normal mode).

It's called the anonymous register because it's the one used when no name is given.
Another name might be the "default" register.

Let's play with it:

- put your cursor on the word "Boban" and do `yiw` (yank inside little word)
- (this should put "Boban" into the `"` register - check with `:echo @"`)
- put your cursor on this "Bobanita Hayworth" and do `di"` (delete inside quotes)
- (this should put "Bobanita Hayworth" into the `"` register - check with `:echo @"`)
- confirm that the `0` (yank) register still has "Boban" in it with `echo @0`
- put your cursor at the end of the sentence below: "And her name was: "
- do `p` to paste from the anonymous register
- press `u` to undo
- do `""p` to paste from the (not very) anonymous register
- press `u` to undo
- do `a` to go into insert mode after the cursor
- do `<c-r>"` to paste in the anonymous register
- press `/` to start a buffer search
- do `<c-r>"<enter>` to start searching for Bobanita Hayworth
- (put back our deleted Bobanita Hayworth)
- put your cursor on the left double quote
- do `p` to paste from the anonymous register

```
Boban
"Bobanita Hayworth"
And her name was: 
```

# Exercise 2 - the blackhole register

This is the `/dev/null` of registers, represented by underscore.

Text sent to the blackhole register goes the way of the dodo (ie. it's gone).

Naturally you can't paste from the blackhole register (it's got nothing in there).

Why would we use this over regular delete? 

- maybe we want to properly delete something and not pollute the anonymous register.

- in the context of scripts and mass deletions, sending text to the blackhole register
  is much faster because it doesn't need to pointlessly copy it somewhere

There's some python code below, naturally our first instinct is to delete it.
Let's make sure we _properly_ delete it though:

- put your cursor somewhere on the code
- do `"_dip` which is:
    - `"_` - blackhole register
    - `d` - delete
    - `ip` - inside paragraph
- confirm that it's not in our anonymous register with `echo @"`
- confirm we can't retrieve it from the blackwhole register with `echo @_`

```python

a = None
def doStuff(b):
  return b + 3
doStuff(a)

```

# Exercise 3 - the expression register

Remember waaaaay back we needed to quickly compute some mathematical expression and we were too lazy to open a calculator.

We wrote a mini python script then used filtering to push it through a python interpreter:

```python
print(42 * 50 - 29)
```

We put our cursor on the line and used `!!python`.
(Here the `!!` is a shortcut for "filter the current line" see [lesson 6](006_operation_shortcuts.md)).

The print statement was necessary because filtering uses the _standard output_ from the program you run,
hence we have to print our result as a side effect rather than return it in the usual FP way.

Overall this is a pretty convoluted way to do a small computation and it requires an interpreter to be installed on our system.

Vim already has a "language" vimscript built into it which supports basic numeric and string operations.

Do `:echo 42 * 50 - 29` and it should print 2071.

What if we want to insert that into our buffer though?

Vim has a special register `=` called the "expression register".
You can send it vim script expressions and it will compute them and give back the answer.

The best way to use this is from insert mode with `<c-r>=`. When you hit `=` you'll see vim prompting you for an expression.

So:

- put your cursor at the end of the line below
- do `a` to go into insert mode after the cursor
- do `<c-r>=` to start the expression register prompt
- do `42 * 50 - 29<enter>`
- (that should cause 2071 to get inserted at the cursor)
- press `<escape>` to return to normal mode then `u` to undo
- do `"=` (paste from the expression register)
- it will prompt you and again type the expression above and hit enter
- then `p` for paste after the cursor

```
42 * 50 - 29 = 
```

Overall dialing up the expression register can be a little disorienting because it will
interactively prompt you for an expression.

The second insertion `"=42*50-29<enter>p` was all one paste command in the form `[REGISTER]p`.
It just seems confusing because getting the value for the register required entering an expression.
