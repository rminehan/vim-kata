# The Dot Operator

Small in stature but a powerful weapon in the right hands.

The dot operator (`.`) repeats the last change you made where changes are things like:

- deleting a word
- inserting some text then returning to normal mode
- indenting a line by 2 spaces

Dot is very useful when you have multiple similar kinds of changes to make.

You make the first change, then move your cursor to a new location and press `.` to apply the change again.

This is easiest to show with some exercises!

# Exercises

## Exercise 1

In the code snippet below, replace all the "Wolfgang"s or "Wolfy's" with "Thilo".

We will do the first replacement manually, then use `.` for the subsequent changes.

- move your cursor to the first "Wolfgang"
- do `ciw` which deletes that word and puts you into insert mode
- type "Thilo" then hit escape to return to normal mode
- (at this point vim would understand your last "change" as being: replace the word under the cursor with "Thilo"
- move your cursor to the next "Wolfgang" and press `.` (the "Wolfgang" should change to "Thilo")
- (note also the period at the end of the line should be preserved)
- move your cursor to the "Wolfy" and press `.` (the "Wolfy" should change to "Thilo")
- move your cursor to the "Wolf" and press `.` (the "Wolf" should change to "Thilo")

```
Wolfgang is our favourite German.

When we need a sassy remark, we go to Wolfgang.

Wolfy doesn't like PHP.

Yesterday we saw the Wolf bring his lunch to work, he's very responsible.
```

## Exercise 2

We wrote some java code but forgot to put semicolons at the end of each line, whoops!
An easy mistake for scala developers who have been liberated from the shackles of semicolonitis.
But we need to fix it to make it compile:

- move your cursor to anywhere on the line starting with "new"
- do `A` to move your cursor to the end of the line and enter insert mode
- type `;` then hit escape to return to normal mode
- (at this point vim would understand your last change as being: jump to the end of the line and insert a ";")
- move your cursor to anywhere on the line with the middle java command (the one with "build" at the end)
- press `.` (this should append a ";" to that line)
- move your cursor to anywhere on the line of the last java command
- press `.` (this should append a ";" to that line)

```java
AbstractProxyBeanFactoryStrategyBuilder myAbstractProxyBeanFactoryStrategyBuilder =
    new ConcreteAbstractProxyBeanFactoryStrategyBuilder()

AbstractProxyBeanFactoryStrategy myAbstractProxyBeanFactoryStrategy = myAbstractProxyBeanFactoryStrategyBuilder.build()

myAbstractProxyBeanFactoryStrategy.launchStrategy(myStrategyProxyDecisionProviderServiceUtilsWrapper)
```

## Exercise 3

We pasted some code into intellij but there was a missing brace which messed up the indentation.

We will create a change to right indent a line by 2 spaces then apply that change many times on various lines.

- move your cursor to anywhere on the line with "howdy"
- do `I<space><space><escape>` (that's capital I then two space characters then escape)
- (`I` puts your cursor at the front of the line in this case and puts you into insert mode)
- (then the two spaces are just to right indent the line by 2 spaces)
- (at this point vim would understand your last change as: jump to the front of the line and insert two spaces)
- move your cursor to anywhere on the next line (the if statement) 
- press `.` to indent the line by 2 spaces
- move your cursor to the next line and press `.` twice to indent it 4 spaces
- carry on indenting the rest of the file based on the commented hints at the end of each line

```scala
def allTheThings(): Unit = {
val a = "howdy" // indent 2 spaces
if (manyThings) { // indent 2 spaces
println("Many things aren't all the things") // indent 4 spaces
} // indent 2 spaces
else  // indent 2 spaces
println("Hopefully this is all the things") // indent 4 spaces
}
```

(Note here we could have also used the `>` operation to indent lines)

## Exercise 4

We wanted to send an angry tweet but realized that not enough of our content was upper cased.

In the text below, upper case all the words like "sad", "huge", "lose" and "fake".

- move your cursor to the first "huge"
- do `gUiw` (that's `gU` to uppercase and `iw` meaning "in word")
- move your cursor to the first "fake"
- hit `.` (this should uppercase it)
- move your cursor to the "lose" on the next line
- hit `.` (this should uppercase it)
- continue on for other words like "fake" and "Sad"

```
I heard on the news that there's so much huge fake news now.

We will never lose, everythign is fake. Sad!
```

# Summary

Often there are many similar changes to be made.

An experienced vimmer will implement that change in a way that makes it repeatable in many locations.

That unleashes the amazing power of the simple `.`.
