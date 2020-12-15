# Formatting

This is just a short one to introduce a little known operator `=` for formatting.

Like our other operators (delete, change, yank), it takes a text object and formats
it based on the rules specific to your file type.

# Exercises

## Exercise 1

Format this C code:

- put your cursor somewhere on the code
- do `yip` to yank the paragraph
- do `:new` to create an empty buffer
- do `:set ft=c` to set the file type to c
- do `P` to paste the code
- do `=ip` (format inside paragraph)
- do `:q!` to kill the buffer without saving it

```c

        #include <stdio.h>
    int main(int argc, char *argv[]) {
int a = 1;
        int b = 2;
            char * buffer = malloc(1024);
    return 0;
}

```

## Exercise 2

(Note I've tested this on a blank neovim started with `-u NONE` and the instructions seem to work.
It just won't be as pretty as a vim with a basic scala plugin intalled)

Format this scala code:

- put your cursor on the line with "object"
- do `V` to go into line-wise visual mode
- press `j` until you've extended the highlight over all the code
- do `y` to yank
- do `:new` to open a new buffer
- do `:set ft=scala` to set the file type to scala
- do `P` to paste the code
- do `set shiftwidth=2` to set the indentation (it might be that already)
- do `gg` to move to the top of the buffer
- do `V` to enter line-wise visual mode
- do `G` to extend the selection to the bottom of the buffer
- do `=` to format the selection
- (:hmm-parrot: still a lot of spaces between words)
- do `:%s/\v\s+/ /g` to replace all occurrences of multiple spaces with a single space
    - `%` - range is the whole buffer
    - `s` - substitute
    - `/` - start search term
    - `\v` - very magic
    - `\s+` - 1 or more whitespaces
    - `/` - start replace term
    - ` ` - a single space
    - `/` - start args
    - `g` - "global", ie. all matches on a line, not just the first
- (whoops that screwed up our indentation!)
- do `set shiftwidth=4` to make it indent with 4 spaces
- do `ggVG` to highlight the whole buffer again
- do `=` to format it with 4 spaces
- (just to be crazy let's use 8 spaces)
- do `set shiftwith=8`
- do `gv` to bring up the last visual selection
- do `=` to format it
- do `:q!` to kill the buffer without saving

```scala

    object    App   {

    def    main   (args:    Array[  String  ]): Unit = 

    {

        println(  "Hello world")

}

 }


```

# Summary

Vim uses the current filetype to know how to format your code.

It will use the extension on the file usually, but you can also explicitly tell it using `set filetype=...`

Vim knows about most common languages and will have basic regex based rules for them
and standard defaults for indentation levels.

There are simple plugins for most languages that will define or improve code formatting rules
and basic syntax highlighting.
