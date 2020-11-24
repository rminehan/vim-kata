# Operations and text objects

Something that makes vim very powerful (and quite unique) is its model of operations and text objects.

Here are some operations:

- y - yank (copy)
- d - delete
- c - change

Here are some text objects:

- iw - inside little word
- ip - inside paragraph

Operations can be combined with text objects in a very powerful way. Just type the key for the operation then the text object.

Examples:

- if you wanted to delete the paragraph your cursor is in, you'd type `dip`, ie. "delete insides paragraph".

- if you wanted to copy the word your cursor is on, you'd type `yiw`, ie. "yank inside little word"

The power of these operations is that they'll work with your cursor anywhere inside the text object.

# Exercises

## Exercise 1

Make a copy of the `accent` method below:

- put your cursor somewhere in the paragraph
- type `yip` ("yank inside paragraph")
- move your cursor to below the method
- type `p` (past)

```scala

def accent(name: String): String = name match {
  case "thilo" => "german-ish"
  case "clement" | "fredrick" => "singlish"
  case "rohan" => "aussie"
  case "paul" => "yank"
  case _ => "<unknown>"
}


```

## Exercise 2

Change "fredrick" to "zij" in the function below:

- put your cursor somewhere on the word "fredrick"
- type `diw` ("delete inside word")
- type `i` to go into insert mode
- type "zij"
- hit escape

```scala
def accent(name: String): String = name match {
  case "thilo" => "german-ish"
  case "clement" | "fredrick" => "singlish"
  case "rohan" => "aussie"
  case "paul" => "yank"
  case _ => "<unknown>"
}
```

This was a little fiddly - we deleted, then manually went into insert mode.

What we really wanted was to "change" Fredrick to Zij (see next exercise).


## Exercise 3

Change Thilo's accent from "german-ish" to "austrian-ish" (not "australian-ish") in the method below:

- move your cursor to somewhere on the word "german" 
- type `ciw` ("change inside word") - this will delete "german" and put you in insert mode
- type "austrian"
- hit escape to return to normal mode

```scala
def accent(name: String): String = name match {
  case "thilo" => "german-ish"
  case "clement" | "fredrick" => "singlish"
  case "rohan" => "aussie"
  case "paul" => "yank"
  case _ => "<unknown>"
}
```

Note here that the "-ish" doesn't get deleted because you're using "little" words.

For extra points you can try the same operation using the "big" word text object: `iW`.

# Summary

If there are N operations and M text objects, then you get roughly N*M possible actions you can take (not all combinations will make sense).

So memorizing N+M concepts gives you roughly N*M things you can do. This is one time where engineers can embrace such unbridled scaling!

The power of vim!

# Plugins

Some plugins introduce new text objects.

Out of the gate those text objects will be usually able to interract with all the operations you already know. You can leverage your existing knowledge of operators with new text objects - vim is great.

Examples are:

- `ii` - lines at my level of indentation
- `ia` - function argument
- `il` - current line

Likewise there are sometimes new operator added. The classic example is Tim Pope's vim-surround plugin (which is a bit hard to succinctly describe here).

# Going further 

There are many other standard operations and text object from standard vim. Here's a few:

Operations:

- `>` - right indent
- `<` - left indent
- `gU` - upper case
- `gu` - lower case

Text objects:

- `iW` - inside big word
- `aw` - around little word
- `aW` - around big word
- `ap` - around paragraph
- `aP` - around big paragraph

Playing with this:

- put your cursor on a paragraph and do `>ip` to right indent the paragraph
- put your cursor on a word and do `gUiw` to upper case that little word
