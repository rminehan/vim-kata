# Wrapping

Here's some vim wrapping:

```
Yo sometimes lines get long - It can feel wrong - Like stinky code it can pong - This is a wrap song not a rap song - I need to `:set wrap` to stay strong - and gj gk can tag along
```

Sadly not everyone in this universe follows the recommended 80 character limit per line...

There's a lot of young people with fancy high res monitors who think it's okay to extend a line to 119 characters wide.

You might also vim to edit prose (ie. waffly/fluffy writing done by non-coders).

So sometimes text will wrap - today is some tools to deal with that.

# Visual lines

There is a difference between "real" lines and "visual" lines.

Real lines are delimited by newline characters and have nothing to do with how the lines visually look. Line based operations (e.g. `Y` - yank current line) are based on the real lines.

However most editors also have a concept of "visual" lines where the rendering of the text in your editor will wrap around based on the width of the editor window and the up/down arrow will be based on visual lines and not real lines (e.g. microsoft word). This makes sense with emails, blogs, readme's etc...

# wrap

Vim has a built in setting `wrap` which controls how lines are rendered (but it doesn't affect most keyboard shortcuts).

Setting it to true (ie. `:set wrap`) makes long lines wrap around your screen, ie. visual lines aren't the same as real lines.

Setting it to false (ie. `:set nowrap`) keeps each visual line corresponding to one real line.

# Modified navigational keys

`j` and `k` move up and down _real_ lines.

When you have line wrapping on, this can be very confusing as you will seemingly jump up and down many visual lines on long line blocks.

`gj` and `gk` are analogous movements which move up and down visual lines. They are analogous to how the up and down arrow keys work in most basic text editors.

Some people think this should be the default and actually rebind `j/k` to mean `gj/gk` respectively.

`^/$` also have visual analogs `g^/g$` for going to the first and last character on a visual line.

# Exercises

## Exercise 1

For this exercise we'll just play with the `wrap` variable:

- turn on line wrapping: `:set wrap`
- turn off line wrapping: `:set nowrap`
- view the state of line wrapping: `set wrap?` (should print "nowrap")
- toggle the state of line wrapping: `set wrap!`
- view the state of line wrapping: `set wrap?` (should print "wrap")

This was also an introduction to the bizarre vimscript syntax associated with vim's built in variables.

## Exercise 2

We'll jump around our wrap song.

This section makes an assumption that your window is narrow enough that the wrap song wraps to at least 2 lines. If you're one of those young people editing this on a 30"+ landscape monitor at some ultra high resolution then you'll need to narrow the window until it wraps around. :old-timey-parrot:

- make sure line wrapping is on: `:set wrap`
- put your cursor on "Yo"
- do `j` - notice how it jumps you down two visual lines
- do `gk` to go up 1 visual line
- do `0` to go to the start of the real line - your cursor should be back on "Yo"
- do `gj` to go down a visual line, then `gk` to go back up
- delete that visual line by doing `dg$` which is:
    - `d` for delete (expecting a text object)
    - `g$` the motion to go to the end of the visual line (which is converted to a text object)
- (there should still be some of the line left over after this)

```
Yo sometimes lines get long - It can feel wrong - Like stinky code it can pong - This is a wrap song not a rap song - I need to `:set wrap` to stay strong - and gj gk can tag along
```

# Personal Style

Most of the time I'm using vim for code, lists or markdown files.

Usually I find wrapping more confusing than helpful (particularly with source code),
so I switch it off by default by setting `:set nowrap` in my vim config.

In markdown files, I'll tend to split paragraphs into multiple real lines,
usually splitting on punctation or logical points in the sentence.
(This file is an exception for teaching purposes - :troll-parrot:)
Mardown renderers will still make it all one paragraph as long as there isn't a blank line between two lines.

For reading markdown files not written in that style though, `:set wrap` can be useful.
