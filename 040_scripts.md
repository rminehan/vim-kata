# Scripts

We're used to thinking of vim as an interactive tool for modifying text files.

Whereas if we wanted some automated process for transforming text, we'd usually think of
using a script (e.g. bash, python).

For example suppose you wanted a general purpose script to print the first 20 lines
in a file that contain "boban" and you had many files to process in this way.
You might write a little shell script:

```bash
cat file | grep boban | head -n 20
```

For simple examples this makes sense.

If we had more complex examples that aligned with the kinds of text objects that exist in vim,
we might want to use a "non-interactive" vim to do that transformation instead.

Just as the python interpreter processes python code, vim has an interpreter for processing vimscript.
It's the language you use in `:` commands and in your vim config files.
You can think of `:` commands as an interactive repl for vimscript.

# Exercises

## Exercise 1

Copy the text below into a file `comment.vim`:

- put your cursor on the normal command in the code block below
- do `:. write comment.vim` which is:
    - `.` - current line as input
    - `write` - send what's on the current line to an external destination (e.g. file or program)
    - `comment.vim` - file
- set a mark here `mx`
- (note the next command will mess with your buffer, you might just run `u` after it then `'x` to return here)
- do `:source comment.vim`
    - this will run the vimscript commands in that file against this buffer
    - that command is an ex command:
        - `%` - the whole buffer
        - `normal` - run a normal command one each line in the range
        - `I#` - go to the front of each line, enter insert mode and insert a '#'
    - it has the effect of commenting out files that use '#' as a comment character (e.g. python, bash)

```vim
% normal I#
```

This is just a simple example to show how to run vimscript files against your buffer.

In reality we'd use tcomment from [kata 39](039_tcomment.md): `gcae` (if it were a python/bash file).
This is less characters and will work for other languages that don't use `#` as a comment.

## Exercise 2

Let's write something more useful for dealing with whitespace. It will:

- remove trailing whitespace from lines
- reduce multiple blank lines to a single blank line
- remove blank lines at the start and end of the buffer

We'll put this in a file `whitespace_clean.vim`:

- put your cursor on the first line of the block below
- do `V` to enter visual line mode
- highlight the entire block (e.g. use `j` to extend down)
- do `:` to enter command mode (vim will add a range for you)
- do `write! whitespace_clean.vim`
    - the `!` here means "overwrite if it exists"
    - not be confused with `write !whitespace_clean.vim` which would try to run a command `whitespace_clean.vim`

```vim
" Remove trailing whitespace
" After this we can assume any blank lines are empty
% substitute /\v\s+$//

" Replace all consecutive newlines with two newlines
" This will compress multiple blank lines together
" if they're in the middle of the buffer, but at the
" start and end it may still leave 2 blank lines
% substitute /\v\n\n+/\r\r/

" Remove the first line if it's blank (twice in case of 2 blank lines)
1 global /^$/delete
1 global /^$/delete

" Remove the last line if it's empty (twice in case of 2 blank lines)
$ global /^$/delete
$ global /^$/delete
```

Now we'll apply it something

- in a similar way to before, highlight the block below
- yank it with `y`
- open a new buffer in a vertical split with `vnew`
- paste with `P`
- do `:source whitespace_clean.vim<enter>`
- it _should_ clean the file up
- do `:q!` to discard the buffer

```


Boban
	Jones


Clement
	ine


	
```

The part of the script for removing blank lines at the start and end of the file might look strange:

```vim
...

" Remove the first line if it's blank (twice in case of 2 blank lines)
1 global /^$/delete
1 global /^$/delete

" Remove the last line if it's empty (twice in case of 2 blank lines)
$ global /^$/delete
$ global /^$/delete
```

You might be asking:

> Why do you need to deal with potentially 2 blank lines?
> Didn't we already compress all the blank lines in the prior step?

The prior commands did compress blank lines but the trick doesn't quite work as expected at the start and end of the file
because of some implicit assumptions it has.
Instead it reduces it to _at most 2_ blank lines.

Hence we run a conditional delete twice.
You might be thinking that we could combine them into one range, e.g.

```vim
1,2 global /^$/delete

$-1,$ global /^$/delete
```

but in the case of a buffer like:

```
# Heading

Blank line above

...

Last line with blank line above
```

this would delete the blank line _after_ the heading and line _before_ the end of the buffer (which is wrong).

What we did here to deal with the possible 1 or 2 lines is clunky but at least it's fairly simple to understand.
There are more clever ways to remove the leading and trailing blank lines that involve using multiline regexes.
Being so tricky they're beyond the scope of this kata but we might revisit this in a later kata on complex regex.

## Exercise 3

Now that we have a handy general purpose script `whitespace_clean.vim`,
we can apply it non-interactively from the command line using `-c`.

We'll create a test file `input` with some dummy text and use vim from the command line to clean it.

Note that for this exercise we're assuming you're using a vim launched from the command line.

- highlight the block below 
- do `:` (which should generate a range)
- add `write! input<enter>`
- open another terminal and do `cat input` to confirm there's a file `input` with the contents we expected
- in that terminal do `[YOUR VIM COMMAND] -c "source whitespace_clean.vim" input`
    - where `[YOUR VIM COMMAND]` is what you've been using to launch vim, e.g. `vim`, `vi`, `nvim`, `gvim` or `v`
- (this isn't quite what we wanted - we're still in vim but we wanted a non-interactive session)
- quit and discard changes with `:q!`
- do `[YOUR VIM COMMAND] -c "source whitespace_clean.vim | wq" input`
    - here we added `| wq` in (write quit)
    - this time it should have modified the file and then quit
- do `cat input` to inspect the file - it should be cleaned

```


Boban
	Jones


Clement
	ine


	
```

The `-c` command lets you run an ex command after loading the file. In our case we did:

```
source whitespace_clean.vim | wq
```

This is vimscript:

- `source whitespace_clean.vim` runs the cleaning file over the current buffer (`input`)
- `|` delimits commands -  analogous to `;` in langauges like java or C (it's not like a bash pipe)
- `wq` saves and quits

The overall effect is a non-interactive session where your file got cleaned - similar to running a bash script.

## Exercise 4

The command from the previous exercise was awkward:

```
[YOUR VIM COMMAND] -c "source whitespace_clean.vim | wq" input
```

Vim has a flag `-s` that lets you source a file as keypresses executed from normal mode.
However our `whitespace_clean.vim` file is a list of vimscript statements, not normal mode keypresses though.

Below is a modified version designed to be run from normal mode.
The main differences are:

- comments are gone
- each command has a `:` before it to put you into command mode
- each command has a `` (enter keypress) after it to execute the command

We'll write this out to `whitespace_clean_normal` (it's not a .vim file as this isn't vimscript) then apply it.

- highlight the text below
- do `:write! whitespace_clean_normal<enter>`
- regenerate the `input` file using the steps from exercise 3 or just manually modify to have some extra spaces
- from your terminal, do: `[YOUR VIM COMMAND] -s whitespace_clean_normal input`

```
:% substitute /\v\s+$//
:% substitute /\v\n\n+/\r\r/
:1 global /^$/d
:1 global /^$/d
:$ global /^$/d
:$ global /^$/d
:wq
```

To put those enter keys in, from insert mode I did `<c-v><enter>`.
More generally `<c-v>` is how you directly put key codes in for characters like escape, enter, and unicode.

# Observations

## Different kinds of scripts

This highlighted that there's two kinds of scripts

- vimscript commands (the ones you type in command mode and put in your vim config)
- normal mode keypress based ones (as if you just started typing the characters immediately after the buffer opened)

To run vimscript commands, you need to do `-c "source [SCRIPT]"` which is a bit awkward to type.
However the scripts themelves are much cleaner.

For normal mode scripts, you can use `-s [SCRIPT]` more directly.

## source vs source!

In exercise 2 we used `source whitespace_clean.vim` to run the vimscript over the current buffer.

There is an analogous command `source!` for sourcing "normal mode" scripts. The `-s` flag above is just a wrapper for it:

```bash
# These are equivalent
vim -s script file
vim -c "source! script" file
```

## Non-interactive mode

To make vim truly non-interactive, you need to add a command to quit.

In exercise 3, we did this by adding an extra `... | wq` ex command:

```bash
[YOUR VIM COMMAND] -c "source whitespace_clean.vim | wq" input`
```

In exercise 4, we _didn't_ do it with `-c`, e.g. we _didn't_ do:

```bash
[YOUR VIM COMMAND] -s whitespace_clean_normal -c wq input
```

Try this out and you'll see it doesn't clean the file.
My guess is that the `-c` arg is applied before the `-s` arg,
which has the effect of quitting the editor before the cleaning can occur.

To get around this in exercise 4 we added `:wq` to our script.
This unfortunately means our script isn't friendly for _interactive_ use with `source!`
as it will close the buffer you're running it against.

# Different vim launchers

There are some situations where I want to start vim with different configurations to the standard one I use, e.g.

- special configurations for plugins
- using certain settings (e.g. `shiftwidth` of 2 instead of 4)

Some contexts where you might want vim to behave differently are:

- coding in a specific language (load some heavy "IDE" style plugins you wouldn't otherwise want)
- git commit messages (maybe have some extra plugins/settings useful there but not otherwise)
- todo lists with specific formatting rules

The `-c` flag is a useful mechanism to achieve this, for example:

```
[YOUR VIM COMMAND] -c "set shiftwidth=2" file
```

Typing this out is tedious though.
If it's something you use a lot, you could create a bash script and put it on your path, e.g.

```bash
#!/bin/bash
# File `v2` - launches vim with shiftwidth of 2
nvim -c "set shiftwidth=2" "$@"
```

(If you're not using nvim adjust the launch command above)

If you want to test it out, save it to `v2` and give it execute permissions:

- highlight the text
- do `:write! v2<enter>`
- do `:!chmod u+x v2<enter>` then `<enter>` to close the output window
    - we haven't covered this use of `!`, just think of it as running an ad-hoc shell command
- in another terminal do `./v2 README.md`
- in that vim session do `:set shiftwidth?<enter>` to confirm it's 2

To be able to launch it with just `v2`, put the file on your system path somewhere (e.g. in `/usr/bin/`).

# vim-better-whitespace plugin

The `whitespace_clean` logic we came up with had a step for removing trailing whitespace on lines.

There's a more general plugin `vim-better-whitespace` that adds a lot of useful whitespace based functionality.

More on this in the [next kata](041_better_whitespace.md).

# Usefulness of vim "scripting"

Overall I'd recommend only using vim for text transformation scripting in private contexts which don't involve other devs.

For example:

- a one off task to clean up a lot of log files
- internal editing optimizations for when you're coding in vim

Here's a few reasons:

## Maintainability

Vimscript is a not a readable language (even to vim users).

Chances are you'll put some obscure command in there which you found on stack overflow to solve some curly problem.

For example a complex regex that uses vim-specific regex tricks, or some obscure ex command.

It made sense as you built it interactively, but later you'll have no idea what it does.

And if it's full of symbols (e.g. `&`, `#` and `$`) then it will be very difficult to search as most search engines
don't support special characters.

Even if it's relatively simple, most developers don't know vim and won't be able to modify it.

The main value of it is seeing the look on their faces when they open the file and are greeted by something like:

```
ggVGgU
```

priceless...

(This just uppercases the buffer btw)

## Portability

Not everyone will have a compatible vim installed.

Even if they do, chances are you'll accidentally create a dependency on some plugin you use that other people won't have.

e.g.

```
gUae
```

This uppercases the whole buffer but assumes you have the `ae` buffer text object installed from [kata 36](036_line_and_buffer_text_objects.md).

Regarding neovim and classic vim, they are gradually starting to diverge too.
Neovim tends to move faster and adds features like lua support that classic vim won't have.

## Ergonomics

As we saw, getting vim to behave in a non-interactive script mode is a bit awkward - it's not really built for it.

## Performance

Vim is still opening a session, and running those commands through its interpreter mechanisms.

If you were doing this in batch for 1000's of files it will be very slow.

# Why might you use it then?

Despite all the reasons above you might still decide to use scripts this way:

- the transformation is very hard to achieve with standard tools like grep, sed, head etc...
- you're working closely with another developer who is an avid vimbecile like yourself and has a similar setup

Related to the first point, some of vim's text objects are very handy and
would be a lot of work to replicate with simpler tools.

Examples of this are paragraphs, little words, or indentation based objects like `ii`.

Likewise some ad-hoc transformations suit macros very well because you have a cursor to keep track of your "location".
This is not something you'll get with a chain of grep and sed commands.

# Best practices

If you _do_ decide to write some scripts, there's some general good practices.

## Add comments where possible

Again vimscript is an obscure, _not_ self-describing, inconsistent language.

Later you'll forget what these cryptic commands do or implicit assumptions to make them work.

## Avoid shortened forms of commands

Shortened forms for commands are time savers for interactive editing, but in scripts they just add unnecessary confusion.

For example use `global` instead of `g` and long forms for the ex commands you're using with global,

e.g. `global /boban/delete` instead of `g/boban/d`.

Likewise with ranges, there are shortcuts that aren't helpful for less experienced vimmers:

e.g. `.,.+3` instead of `,+3` (don't drop the `.` even though you can).

Likewise space out ex commands:

e.g. `:% substitute /boban/enxhell/g` instead of `:%substitute/boban/enxhell/g`

The extra keystrokes won't add much time for you, but will help people out a lot later.
Remember code is typically read more than it's written so optimize for the reader.

## Try to avoid noise/pollution to the state of the editor

Often developers have state in their head, e.g.

- the last text I yanked
- the last search I ran
- the last insert I made
- the last text I highlighted

If your script starts yanking or highlighting things, it will mess up these registers and marks.

Sometimes this is unavoidable or just a tradeoff we make.

But there are a few little things you can do to lower the impact.

For example when doing a substitute, prepend the `keeppatterns` command, e.g.

```
:keeppatterns % substitute /enxhell/boban/g
```

This will prevent the search term "enxhell" getting added to the search history.

Another example might be that you intend to yank some text.
Instead of yanking to the default register `"` you could yank to another register like `m`.
There is still a chance the user will be using this register, but it's much less likely than the default register.

There's more to be said here and it's beyond this kata - the message here is just to be aware of it
and take reasonable steps when the cost/benefit is worth it.

## Avoid dependencies on non-standard settings/plugins where possible

If you're going to be passing scripts around to be run from the cli, then design them to work with "vanilla" vim.

You can run vanilla vim by passing `-u NONE` when you start it, e.g. `nvim -u NONE`.

Internally when this starts vim, it doesn't read from any config, so it _should_ be the same for most people
regardless of personal config provided you have similar versions of vim installed.

# Conclusion

When we look at vim as an interpreter (ala python and bash), then vim becomes a text transformation tool we can use:

- interactively by sourcing scripts with `source` and `source!`
- non-interactively from the command line using `-c` and `-s`

This can be used to do ad-hoc text transformations that are too hard with standard tools like grep and sed.

However it often doesn't make sense to use vim this way, so make sure it's really justified to save yourself a lot of pain.

If you decide to use vim in this way, make sure to follow best practices to reduce the pain.
