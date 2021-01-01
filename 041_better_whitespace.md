# vim-better-whitespace plugin

In the previous kata we wrote a vimscript for doing whitespace cleaning.

There is in fact a really useful whitespace related plugins [vim-better-whitespace](https://github.com/vim-scripts/better-whitespace):

```vim
Plug 'ntpeters/vim-better-whitespace'
```

Install it to do this kata.

# Nifty commands

- show trailing whitespace on lines - turn on/off with `:ToggleWhitespace`
- remove all trailing whitespace on lines - `:StripWhitespace`
- remove all trailing whitespace on dirty lines - `:StripWhitespaceOnChangedLines`
- automatically strip whitespace on save - turn on/off with `:ToggleStripWhitespaceOnSave`

# Terminate trailing whitespace with extreme prejudice

Trailing whitespace is mostly just stylistically annoying, but now and then it can be a source of subtle bugs.

## Bugs

For example if you have multiline literal strings in your code like:

```scala
val buggyString = """The 
                    |Big
                    |Boban""".stripMargin
```

Depending on how your logic works, that sneaky space after "The" could be an issue.

## Git pollution

Also if you let trailing whitespace into new code your write,
then later someone else (or _something_ else like an automated formatting/linting tool)
will remove it.

That extra commit to remove the whitespace was avoidable noise in your commit history.

When someone uses `git blame` or uses the blame view in their editor, they'll see that whitespace commit
rather than the one prior to it.

They can of course go back further with more complex use of blame, but it's an extra annoying step.

## Exterminate!

So I prefer to just always exterminate trailing whitespace if it doesn't have a justified reason to exist in the code itself.

# Atomic commits with StripWhitespaceOnChangedLines

The `StripWhitespaceOnChangedLines` is really useful for keeping your commits atomic,
ie. not mixing refactoring with logical changes.

If you're halfway through modifying a file and you decide to run `:StripWhitespace`,
that will create a big noisy whitespace diff that will obscure the logical code changes you made
and you'll make life harder for your code reviewer.

It also increases the chances of a conflict as it might have a big change footprint across your file.

But maybe you still want to strip the whitespace off the code you've modified anyway.
`StripWhitespaceOnChangedLines` will only clean up the lines that you've modified.

Note this functionality probably depends on some git plugin like vim-fugitive under the hood.
I can't find anything specifically mentioned on the readme though.

# Exercises

## Exercise 1

This exercise is to play with the `:ToggleWhitespace` setting

- do `:ToggleWhitespace` to turn whitespace delection on (or off)
    - (you should see whitespace detected after "He's" and "Boban" in the block below)
- do `@:` as a quick way to repeat the last command
    - this is a special example of "play the macro in the register"
    - here `:` represents the "last command" register

```
He's 
a real
Boban 
```

## Exercise 2

For this exercise we'll remove some whitespace from a subsection of the block below.

`StripWhitespace` takes a range (defaulting to the whole buffer if none is specified).

- toggle whitespace detection on
    - you should notice that the odd numbered lines below have trailing whitespace
- put your cursor on line 0
- do `:.,+3 StripWhitespace<enter>`
    - this should have removed whitespace from 1 and 3, but not 5
- hit `u` to undo
- with your cursor again on line 0 do `:.,+6 StripWhitespace<enter>`
    - all whitespace should be removed
- hit `u` to undo
- with your cursor again on line 0 do `:.,+6 StripWhi<tab>`
    - this is just to demonstrate tab completion
    - hit enter or escape to leave command mode

```
0
1  
2
3  
4
5  
6
```

## Exercise 3

Let's take advantage of the fact this file is under version control and use `StripWhitespaceOnChangedLines`.

For this exercise we'll add whitespace to some lines and run the command above.
It should only affect lines we've modified and not other whitespace trailing lines.

- make sure whitespace detection is on with `:ToggleWhitespace`
- for the lines below that start with "change me", add some whitespace to the back, e.g. `A<space><escape>`
    - but don't touch the other lines
- run `:StripWhitespaceOnChangedLines<enter>`
    - we're running it over the whole buffer
    - notice how it only stripped whitespace on the changed lines

```
NO TOUCHING   
NO TOUCHING	
change me
change me
NO TOUCHING   
change me			
```

If this doesn't work, chances are you need a git plugin like vim-fugitive.

# Conclusion

Trailing whitespace ranges from annoying to dangerous and it makes sense to just exterminate it.

This is a great plugin for dealing with whitespace.

Today we just played with a few common operations. Check out the readme on the plugin for more details.
