# Auto-complete

Most developers feel a bit helpless without auto-complete.

Today we're not looking at language specific auto-complete, but simpler generic forms of auto-completion:

- buffer word completion
- line completion
- filename completion

These don't replace the language aware auto-completion you get with sophisticated editors but are still useful.

# Shortcuts

These menus appear in insert mode.

`<c-x>` is the first key press for bringing up these simpler vimmy auto-complete menus:

- `<c-x><c-n>` - buffer word completion
- `<c-x><c-l>` - line completion
- `<c-x><c-f>` - filename completion

Depending on the plugins you have installed and general config, _how_ the auto-complete menu is rendered
and what triggers it to appear might be slightly different.

Once it appears, you should be able to use `<c-n>` to cycle forward through results and `<c-p>` to cycle backwards.
Think of "n" for next and "p" for previous.
This matches bash's history navigation so should feel fairly intuitive.

If you used `<c-x><c-something>` to bring a menu, keeping `<c-x>` suppressed and hitting `something` should also
cycle through results.

If you decide you don't like any of the options use `<c-e>` to go back to the text you
had before dialing up the auto-complete.

# Exercises

## Exercise 1

Our buffer has some Bobany words:

```
Bobanita
Bobanish
Bobanson
```

Use this to do word-complete the sentence below to "They called him Bobanson":

- put your cursor on the line
- hit `A` to go to the end of the line and enter insert mode
- do `<c-x><c-n>`
- hit `<c-n>` until "Bobanson" is selected
- hit escape to go back to normal mode as there's nothing else to type

```
They called him Bob
```

## Exercise 2

Here are some Bobany sentences:

```
Boban the mighty
Boban the conqueror
Boban the Bobanish
```

Use this complete the sentence below to "Boban the Bobanish"

- put your cursor on the line
- hit `A` to go to the end of the line and enter insert mode
- do `<c-x><c-l>`
- hit `<c-n>` or `<c-l>` until "Boban the Bobanish" is selected
- hit escape to go back to normal mode as there's nothing else to type

```
Boban the
```

## Exercise 3

In this directory we have lots of kata files.

I'll leave it to you to pick your favourite one to complete the sentence below.

- put your cursor on the line
- hit `A` to go to the end of the line and enter insert mode
- do `<c-x><c-f>`
- hit `<c-n>` or `<c-f>` until you've picked the one you like
- hit escape to go back to normal mode as there's nothing else to type

```
My least unfavourite kata so far is 0
```

## Exercise 4

Filename completion also works with absolute paths and `~` style paths.

Use this to complete the sentence:

```
My vim kata directory is in ~/
```

# Conclusion

There's a few more simple built in auto-completions but these 3 are a good start for now.
Other ones include dictionary completion (useful for words you keep mispelling/misspelling)
and omnicompletion which searches for words outside the current buffer.

During the creation of these kata I used linewise completion a lot because there is a lot
of repetition of instructions between katas.
I've also found that when writing tests there are often small sections of code repeated in different
tests (e.g. simple setup code) and linewise completion works is very useful.

Filename based completion has been very useful during these kata because of the cross referencing of kata's.
It's great for markdown files and cross linking.
The paths are relative to your working directory which works well for projects with simple hierarchies,
but not complex maven/sbt projects with complex structures.

People who use vim for heavy coding will use sophisticated frameworks like coc to support much more complex
auto-completion frameworks, e.g. the language-server-protocol (LSP) and tab-nine.
These frameworks will often automatically bring up the auto-complete menu and let you use the tab key to navigate
through the results.
Heavy use of these tools is sometimes referred to as "dark vim" because some people feel it's too heavyweight
and was not what vim was designed to be used for.
