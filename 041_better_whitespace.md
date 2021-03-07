# vim-better-whitespace plugin

In the previous kata we wrote a vimscript for doing whitespace cleaning.

There is in fact a really useful whitespace related plugins [vim-better-whitespace](https://github.com/vim-scripts/better-whitespace).

It provides ways to visually detect trailing whitespace and remove it.

# Exercises

So do the exercises you'll need the kata installed.

```vim
Plug 'ntpeters/vim-better-whitespace'
```

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

# Summing up the plugin

It's got these commands:

- show trailing whitespace on lines - turn on/off with `:ToggleWhitespace`
- remove all trailing whitespace on lines - `:StripWhitespace`
- remove all trailing whitespace on dirty lines - `:StripWhitespaceOnChangedLines`
- automatically strip whitespace on save - turn on/off with `:ToggleStripWhitespaceOnSave`

# Exterminate!

Generally I feel that trailing whitespace is a problem and as a rule of thumb I'd delete it (which this plugin is great for!).

Before we send in the daleks, I should give you my reasons:

## Bugs

It's easy for sneaky trailing whitespace to cause subtle logic bugs.

For example in this multi-line string, there's a space after "The". Depending on what's processing this string, that could cause an issue.

```scala
val buggyString = """The 
                    |Big
                    |Boban""".stripMargin
```

## Git pollution

If you let trailing whitespace into new code your write,
then later someone else like me (or _something_ else like an automated formatting/linting tool)
will remove it.

That extra commit to remove the whitespace will mean that the last commit that touched that line will be a whitespace removal commit,
rather than a _logical_ change to that line (which is usually what we're intersted in).

When someone uses `git blame` or uses the blame view in their editor, they'll see that whitespace commit rather than the one prior to it.
They can of course go back further with more complex use of blame, but it's an extra annoying step that requires deeper knowledge of git
and not all IDE's that integrate with git will necessarily support that well.

# Atomic commits with StripWhitespaceOnChangedLines

If you're halfway through modifying a file and you decide to run `:StripWhitespace`,
that will create a big noisy whitespace diff that will affect areas of the file you didn't change.

You're effectively mixing whitespace changes with whatever unrelated changes you were originally making which is bad git hygiene.

The `StripWhitespaceOnChangedLines` is really useful for this situation because it will only affect the currently dirty lines.

Note this functionality will require some kind of git plugin so that vim knows which lines are considered dirty.
The docs don't make it clear which plugin it uses but I suspect it's vim-fugitive.

# Conclusion

Trailing whitespace ranges from annoying to dangerous.

There's good reasons to exterminate it and generally no good reasons to keep it, so a simple rule of thumb is to just exterminate it.

This plugin is great for dealing with whitespace. It will visually show it to you and provide quick tools to eliminate it.

Today we just played with a few common operations. Check out the readme on the plugin for more details.
