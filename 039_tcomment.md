# tcomment

tcomment is a plugin which defines "comment" operators for commenting/uncommenting source code.

You can use it with existing line based text objects for commenting out chunks of code -
another win for operator + text object!

The plugin is [here](https://github.com/tomtom/tcomment_vim).

# Operators

The plugin comes with 3 operators:

- `gc` - toggle comments
- `g>` - comment
- `g<` - uncomment

The comment toggling operator will try and work out what kind of block you have.
If it decides it's commented, it will uncomment it.
If it decides it's uncommented, it will comment it.

If the commenting is mixed it in that block though, it might guess the wrong way,
hence it's handy to have the others so that you can be explicit, and you might prefer to just always use them.

# Line based

Following in the vim tradition, doubling the operator `gcc` applies it to the current line.

Unfortunately that's not the case for `g>` and `g<` though.
For those `g<c` and `g>c` apply them to the current line.
So basically the rule is "tack on a 'c' to make it apply to the current line".

# Exercises

Through these exercises we'll be messing with the filetype to change the logic for commenting.
This will make the markdown formatting break.
To change the filetype back to markdown do `set ft=markdown` (or just restart vim if that doesn't work).

## Exercise 1

- set the filetype to C: `set ft=c`
- put your cursor on the first line and do `gcip` which is "toggle comment inside paragraph"
- press `.` to apply the operation again (which should toggle commenting off)
- do `gcc` to comment the current line
- do `g>ip` to comment the paragraph which will probably double comment the current line

```c

/* int a = 1 */
int b = 2

```

## Exercise 2

For this one we'll recap some of the indentation operators from [kata 37](037_indentation_text_objects.md).

- set the filetype to python: `set ft=python`
- put your cursor on the `bar` or `baz` line
- do `gcii` to comment out the body of the function
- do `.` or `u` to undo the commenting
- do `g>ii` to comment out the body again
- do `g<c` to just uncomment the current line
- do `g>ai` to comment out the whole method

```python
a = 1
def foo(b):
    bar(b)
    baz(b)
```

## Exercise 3

- set the filetype to markdown: `set ft=markdown`
- (the next command will mess up the buffer, just remember to hit `u` after it to reverse it)
- comment out the entire buffer with `gcae`
- hit `u`

# Conclusion

Vim shows its power again by letting us apply a new operator to our existing text objects.

Commenting is naturally very useful for coding and this nifty little plugin provides a vim operator for that.
