# The "normal" command

The vim "command line" (triggered by typing `:`) has a very powerful command "normal".

It allows you to apply a change to many lines of code.

Examples of where this would be useful:

- commenting out a batch of lines
- indenting a batch of lines
- clearing a batch of lines

The general syntax we'll play with is: `[RANGE] normal [NORMAL MODE COMMAND]`

It applies the normal mode command to each line in the range.
For example `10,20 normal Afoo` will act as if for lines 10 through 20, we started in normal mode and then ran `Afoo` then left normal mode.
The effect is that all the lines will have "foo" appended to them.

Simple examples of ranges are:

- `%` - the entire file/buffer
- `.` - the current line
- `3,5` - lines 3-5 inclusive
- `'<,'>` - the last visually selected range

# Exercises

## Exercise 1

Remember the Java code from [kata 1](./001_dot_operator.md) that was missing semicolons?

Here it has no blank lines and each command is on a single line which makes it suitable for a normal command.

- determine the start and end line numbers
- (let's assume they're 39 and 41)
- do `:39,41 normal A;<enter>`

```java
AbstractProxyBeanFactoryStrategyBuilder myAbstractProxyBeanFactoryStrategyBuilder = new ConcreteAbstractProxyBeanFactoryStrategyBuilder()
AbstractProxyBeanFactoryStrategy myAbstractProxyBeanFactoryStrategy = myAbstractProxyBeanFactoryStrategyBuilder.build()
myAbstractProxyBeanFactoryStrategy.launchStrategy(myStrategyProxyDecisionProviderServiceUtilsWrapper)
```

The command means:

- for each line from 39 to 41,
- start in normal mode on that line somewhere
- then do `A;` (which moves the cursor to the end of the line, goes into insert mode, then writes a ";") 

Note that you can undo this entire operation in one step with `u`.

## Exercise 2

Someone wrote some python code. Quickly comment it out before it breaks something!

- move your cursor to anywhere on the first line (a = 3)
- do `:.,.+5 normal I#<enter>`
- to uncomment it, do `:.,.+5 normal ^x<enter>` from the first line

```python
a = 3
b = 10
c = 30
def foo(d):
  return a + b * c - d
foo(None)
```

That weird vim incantation will look strange and confusing.

Let's break down the first one:

- the `:` is just to enter command mode
- the next part `.,.+5` is a range in the form of `start,finish`
    - the start is `.` which means "current line"
    - the end is `.+5` which means "current line + 5" (which is the line `foo(None)` is on)
- then there's `normal` as usual
- `I#<enter>` is the normal command
    - `I` means move the cursor to the start of the line and go into insert mode
    - `#` will cause a "#" to be entered (which is how you comment out python code) 
    - `<enter>` is just to submit the command

The second one is the same except the normal command is `^x`:

- `^` means move the cursor to the first non-whitespace character on the line
- `x` means delete the character the cursor is on
- the effect of this is to remove the "#" we previously inserted 

(Note that vim has more sophisticated ways to comment code - there is a plugin which adds a comment operator `gc`)

## Exercise 3

Upper case the first little word on every line in this file:

- do `:% normal ^gUiw<enter>`
- hit `u` to undo!

Breaking this down:

- the `:` is just to enter command mode as usual
- the next part `%` is a range which means "the whole buffer"
- then there's `normal` as usual
- `^` means move to the front of the line (technically the first non-whitespace character - see also `0`)
- `gUiw` is `gU (uppercase) + iw (in little word)`, ie. upper case the little word the cursor is on
- `<enter>` submits the command

# Summary and Comparison with '.'

Use the normal command when you want to apply the same change to a contiguous block of lines.

The dot operator overlaps with the normal command in that you specify the command once and then can reapply it by moving your cursor and pressing dot.
It works better when you need to reapply your changes in a more ad-hoc way (e.g. multiple times in a line, or on disconnected lines).
