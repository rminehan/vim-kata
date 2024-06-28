# Advanced Regex 9 - Search shortcuts

Today's kata is a miscellaneous collection of tips and shortcuts to build search and replace operations more quickly.

To clarify, I don't mean that the searches themselves execute faster, but the process of building them is faster.

# Exercises

See the [setup guide](advanced_regex_exercises_setup.md).

Also for this exercise we'll turn _off_ `inccommand` as we're using `~` a fair bit:

```vim
" Type this to set inccommand to an empty value (effectively disabling it)
:set inccommand=<enter>

" Confirm it's empty
:set inccommand?<enter>
```

You might also want to disable search highlighting as it creates a lot of noise in the examples
when we search common characters like 'A'.

```vim
:set nohlsearch
```

The appendix explains a lot of these obscure commands in more detail. We're going in head first here!

## Exercise 1 - many replacements

We're going to do use `substitute` in many different ways to replace 'A' with 'B' on a line of text.

Each instruction is paired with the text to transform in a form like:

```
INSTRUCTION 1       (COMMENT)
TEXT 1

INSTRUCTION 2
TEXT 2

...
```

Some instructions related to fzf and `<c-f>` are too big to fit on one line so they spill.

For each instruction, we're leaving out the range which defaults `substitute` to just the current line.
So run each command with your cursor _on the line_ being transforming.

```
:s/A/B/g<enter>      (Standard replacement, no tricks here)
AAABBB

:s//B/g<enter>       (Leaving the search blank, it will use the search from the previous substitute)
AAABBB

:s/A/~/g<enter>      (Using tilda which will be 'B' due to the previous replace we just ran)
BBBAAABBBAAA

:s//~/g<enter>       (Combination of the two above)
AAAXXX

:<c-f>               (Using the handy `<c-f>`)
[PUT YOUR CURSOR ON ONE OF THE ABOVE]<enter>      
ABAB

:History:<enter>     (Using fzf's fuzzy searching of command history)
[TYPE s//~ TO NARROW A MATCH]<enter>  
aAbBcC

&&&                  (`&` from normal mode replays the last substitution, but with no flags, so we have to use it 3 times)
AAA

:s<enter>:s<enter>   (`&` is just an alias for `:s` - see `help &`)
AA

:s#A#B#g<enter>      (Using '#' instead of '/' as delim)
AAABBB

:&&<enter>           (Like `&` but remembers the flags. It's replaying the `s#A#B#g` search above which used `g`)
AAA   AAA

@:                   (Replay last ex command)
AAAyyy

:s<enter>5@:         (Using `:s` version which is an ex command so that we can use a count with `@:`)
AAAAAA

:History:<enter>     (We're modifying an existing command with <c-e> to be case insensitive, should replace 'a' too now)
[TYPE '###' ENOUGH TO NARROW TO s#A#B#g]
<c-e>[ADD `i` FLAG, ie. s#A#B#gi]  
aAaAbB

:<c-f>               (Modify our command history so that the previous one is listed with `c` for interactive replacement)
[FIND THE PREVIOUS COMMAND AND APPEND `c`, ie. s#A#B#gic]
<enter>
aaAAaaCC

q:                   (This brings up the <c-f> window for ex command history - it doesn't quit vim, that's `:q`)
[SELECT A PREVIOUS REPLACEMENT]
<enter>
AAaaBBbb

3&kk3&kk3&kk3&             (Adding a COUNT to `&` makes it still only apply the change once, but now on a multiline range)
AAAA
AAAA
AAAA
```

## Exercise 2 - breaking down a line

We'll use `&` to gradually chop down a long comma separated string.

- put your cursor at the start of the string
- determine the line number
- do `/\v,@<=\s*<enter>` to test out searching space following commas on the line of the cursor

```
token 1, token 2,token 3,  token 4, token 5,token 6,token 7
```

If it's working, you should be able toggle through the matches with `n` and it will jump around to the position
just after the comma on each match.

Now we have our pattern, we can start to split the line up:

- put your cursor on the line somewhere
- do `:s//\r<enter>`
    - recall leaving the pattern empty means to reuse the last search which you can see with `:echo @/<enter>`
    - this replaces the first match with a carriage return (`\r` here is "carriage return")

If all worked well, there should be two lines and your cursor will be on the second line.

```
token 1,
token 2,token 3,  token 4, token 5,token 6,token 7
```

- press `&` a couple of times
- do `:s<enter>`
- do `2@:`

Each token should be on its own line now.

If you _didn't_ want to remove the whitespace at the start of each token, you could use `\v,@<=` for your pattern.
It matches any position after a comma, but doesn't consume any characters, so it's zero width.

We could have done this all in one go using `/g` on our first replacement.
But perhaps in a case like this you want to do it interactively until the line is short enough.

## Exercise 3

The odd thing about the previous example was that we effectively chopped off the front of the long string.

Usually when you're breaking down long lines, you leave the longer part on the line above and the shorter part
goes below, e.g.

```
# Before
This is a really really really really long long line

# After
This is a really really really really long
long line
```

We'll use a greedy `.*,` pattern to find just the last comma on the line.
It will run to the end of the string then backtrack until it finds a comma.
We'll then replace the `\s*` whitespace after it with a newline like above.

After each chop, the last comma will keep moving back toward the front of the string as we apply `&`.

- do `/\v.*,\zs\s*\ze<enter>`
- put your cursor at the start of the string
- do `gn` then escape to confirm it matches the whitespace after the last comma
- put your cursor somewhere on the line and do `:s//\r<enter>`
- do `k` then `&` to chop again
- whoops! It put in an empty line...
    - this is because it matched the comma after token 6
- delete that line with `dd`
- do `/\v.*,\zs\s*\ze$@!<enter>`
    - note the negative lookahead `$@!` saying "there can't be a line end after the `\s*`"
- with your cursor on the first line, do `:s//~<enter>`
- do `k` then `&`

```
token 1, token 2,token 3,  token 4,   token 5,token 6,  token 7
```

By the end you should have:

```
token 1, token 2,token 3,  token 4,
token 5,
token 6,
token 7
```

At this point maybe you've decided the line has been chopped enough.

You could join the shorter ones together by putting your cursor on "token 5" and doing `3J`.

### Quick tangent:

These are the patterns from exercise 2 and 3:

```
# Exercise 2
\v,@<=\s*

# Exercise 3
\v.*,\zs\s*\ze$@!
```

Here's the cores of the pattern aligned for comparison:

```
# Exercise 2
    ,@<=  \s*

# Exercise 3
.*  ,     \s*
```

In exercise 2 we used a positive lookbehind: match if I'm after a comma

In exercise 3 we didn't do that. We actually consumed the comma and characters before it
and used `\zs` and `\ze` to restrict the matched text to just `\s*`.

This is because we wanted to match the last comma.
To get the last comma we needed to use something greedy to run to the end of the string and backtrack
until hitting a comma.

If we did `(.*,)@<=`, then that means: match if there's any text then a comma _before_ me.
That will match every position just after a comma, because the algorithm is looking back.
We'd end up chopping from the front.

To get to the end of the string, we need to let `.*` consume characters forwards.
Hence we went with a strategy of just consuming characters and then using `\zs` and `\ze`
to narrow the match to just the `\s*` we want to replace.

## Exercise 4 - playing with tilda

Usually tilda is used in the replace section of a `substitute`, but we can also search with it.

- enable `hlsearch` with `:set hlsearch<enter>`
- highlight the below
- do `:s/\va{3}/x/g<enter>` (this replaces triplets of 'a's with 'x')
- do `/\v%V`
- add `~` to find all replacements
- add `b` to find all replacements before a 'b'
- replace the `b` with `~` to find cases where two triplets got replaced back to back

```
aaa
aaa
ab
aaaaaa
aaaaaab
aaac
```

Turn `hlsearch` off again if you're going to do the next exercise.

## Exercise 5 (optional) - inccommand madness

In the warmup, we mentioned how `inccommand` can ruin `~` because it keeps updating its value whilst you
interactively type your command.

Just for fun we'll do some slo-mo interactive editing to understand the crazy stuff going on.

Do `set inccommand=split<enter>`.

You also might want to split the window with `:split` or `:vsplit` as the instructions run down a fair way.

Highlight the below and type out `:s/~//g` (without hitting enter at the end).
Regardless of what `~` previously meant, it's now the empty string because that's what our replacement is.

It will match at every position, but those matches have no width.

```
ab
a
b
```

So `:s/~//g` is telling vim to replace all those zero width matches with nothing.
Overall that has no effect.

Insert an `a` for the replace text (`:s/~/a/g`).
For me at least, the meaning of `~` doesn't update yet - it will take _another_ keypress.
So at this point `~` is still the empty string.

As mentioned above, the empty string will match with zero width at every position.
So now we're replacing all those zero width matches with 'a'.

There is a zero width match before every character.
So you should see 'a's appear before every character as vim shows you what it would look like to replace
every zero width match with an 'a'.

Add `b` to the replace text (`:s/~/ab/g`),
Hitting `b` changes `~` to update to `a` (the last used replace).
Now every 'a' is being replaced by 'ab'.
You should see line one become "abb" and line 2 become "ab".

Lastly if you then add `c` (`:s/~/abc/g`)
then the tilda updates to become `ab` meaning every "ab" changes to "abc".
You should see line one become "abc".

Overall it's very confusing and the original value of `~` you were wanting to use is long gone.

If you use a different sequence of key presses to get to `:s/~/abc/g` you might get something else.
I'm sure there's an academic paper for Enxhell to write about leveraging these recursive replacements.
Or a potential computer game to replace the text with as few keystrokes as possible.

So once again, if you plan to use `~`, then disable `inccommand` by doing `:set inccommand=<enter>`.
If it's in the replace section then you can probably get away with leaving `inccommand` on.

# Conclusion for today

Today was about miscellaneous little shortcuts mostly around executing `substitute` commands faster.

They're a bit old school and might not be so useful now given we have `inccommand` and fzf's `History/` and `History:`.

# Conclusion to advanced regexes

That's it for looking at vim's advanced regex syntax.

Some of what we covered is just advanced regex concepts you'll find in other contexts
(like look arounds and back references)
so it should be useful beyond vim.

Some of it is quite useful for building buffer transforming scripts you can run with `:source`.
For example multline patterns and programmatically building commands work well in those contexts,
but they're not things you'd use in day to day editing.

We also looked at vim specific regex concepts like the `{a,b}` (lazy) range, and special vimmy atoms
like `\k` for keyword and `\i` for identifier.

There's a lot here and if it doesn't stick it doesn't matter that much,
you just need to have a vague memory that the concept exists, then use the cheat sheet to find the details.

Also learn to use vim's built in help.
When searching for pattern based stuff, usually it's `:help /` then the thing you're searching, 
e.g. `:help /\%#` for matching the cursor.

If you find regex interesting and want to understand where vim's engine sits in the grand scheme of things,
[this](https://en.wikipedia.org/wiki/Comparison_of_regular-expression_engines) is a good article.
In particular you can see how perl has a lot of features vim doesn't.
If you want to try and use perl regex within vim, look into `:perl` and `:perldo`.

# Appendix

# Empty search term

If you run a buffer search or `substitute` with an empty search term, the value of the `/` register is used.

Recall that the `/` register gets set when you run a buffer search or a `substitute`.
Basically it represents the last search term used.

For example if you searched `/abc<enter>` then did `%s//ABC/g`, the `substitute` will replace "abc" with "ABC".
Then doing `/<enter>` would search again for "abc" (probably finding nothing in this case as they were all replaced!)

This is helpful because you often first test out a search pattern before applying a substitution.
Instead of having to copy the pattern into the `substitute`, you can effectively decouple search and substitution.

Practical Vim discusses the pro's and con's of this in tip 91. Here's a few notes:

One of the downsides of this approach is that later if you went through your command history,
you wouldn't know what text you substituted.

It does work well though if you're wanting to apply the same substitution to different search patterns.
You'd check each search pattern using `/` then can use the same `s//...` on each.

If you don't like this approach, you can continue using `<c-r>/` to insert the contents of the search register
whilst building your substitution.
It's a couple of extra key presses but does leave your history very clear.

I think the mindset for the "search first then substitute" came from there being no way to inspect ahead of time
all the matches you'd get during your substitute.
Now that vim has `inccommand` you actually can interactively view the effect of a substitute before committing to it,
so that lessens the need for these kinds of tricks.
Note as well that `substitute` has a `c` flag for interactively prompting you whether to replace each item.
Still by this point though you've already locked in the search term so it's too late to fix it.

# History in a buffer

Often you want to dial up previous searches or commands, or even yank them into a register.

Once you're in search or command mode, there is of course `<c-n>` and `<c-p>` for navigating back and forth
through your history but it's hard to see.
Sometimes you want to search your search history!

However vim also provides a mechanism `<c-f>` which opens the history in a buffer.
You can then use the full power of vim's modal editing in that buffer (e.g. searching it, modifying commands).
This gives you a chance to run modified versions of previous commands.

One subtle difference between this mode and normal is that when you press enter,
it runs the entry your cursor is on.

Note that changes you make in that buffer don't actually modify your history.

As usual, use `:q` to close the buffer.

Note that if all you want to do is search and run previous commands, then the fzf plugin covered earlier
is probably better for this because of its neat fuzzy searching.
For example `:History:` and `:History/` correspond to command and search history respectively.
You can even use `<c-e>` to select an item for editing, without executing it.

## Confusion quitting vim

Sometimes when you try to do `:q` you accidentally do `q:`.

It turns out `q:` is equivalent to hitting `<c-f>` from command line mode.

So instead of quitting vim, you end up with another buffer open full of obscure looking commands.

This adds to why "How to quit vim" is one of the most upvoted questions on stack overflow.

# Reusing replacement strings

After you do a replacement, `~` takes on the meaning of the replacement text.

For example if you ran `s/foo/b/` then ran `/a~c`, you'd be searching "abc" as `~` is 'b'.

It works in replacement text too. After the above, if you did `s/bar/~az/`, it would replace "bar"s with "baz".

Some use cases for this:

- checking through all the replacements after a `substitute` command (`/~`)
- running a follow up `substitute` with a different search pattern, but the same replacement (`s/.../~`)

## Issues with inccommand

In our [setup file](advanced_regex_exercises_setup.md) we covered the useful `set inccommand=split`
to make `substitute` interactive.

Unfortunately this completely messes up the meaning of `~` (the last substitution)
because the interactive nature of the search keeps updating it whilst you build your substitute command.
By the time you've entered `:s/` the value of tilda is already the empty string.

This makes for a very confusing experience and it makes sense to turn off `inccommand` if you're planning to
interactively build `substitute` commands using `~`.

To disable it, set it to an empty value: `:set inccommand=<enter>`.

# Redoing a replace

From normal mode, `&` repeats the last substitution but applied to the current line with no args (see `:help &`).

For example if we'd run `%s/A/B/gi` as our last substitution somewhere,
then hitting `&` will run `s/A/B` against the current line.

Note the missing `gi` from the args and the range being excluded (which defaults to the current line).

The effect is the first 'A' on the current line will change to 'B'.
If you had 3 A's on your line to replace, you'd hit `&` 3 times.

Overall `&` is like a localized/granular substitution which might make sense in contexts where you want to do
one replacement at a time in a more controlled way.

## Repeatability?

Unfortunately `&` doesn't count as a "change" which `.` recognizes.

So to repeat it you need to just keep hitting `&`.

## Counts?

Doing `[COUNT]&` doesn't apply `&` count times.

Instead it applies the substitute over many lines in the same way as for the indentation operators `<` and `>`.

For example `3&` is like running:

```
.,.+2 s/.../~/
```

You'll get up to one replace on each line.

## How to apply it many times then?

You can keep hammering `&` which isn't too hard.

It turns out that `&` is the same as running `:s` and `:s` is an ex command
which we _can_ repeat with `@:` and further we can use a count with `@:`.

So if you want to apply it a certain number of times, you could do `:s<enter>` then `[COUNT]@:`
to pick up the remaining replacements.

If you want to substitute all matches on the current line, then just do something like `s//~/g`.

Here we're leveraging knowledge from previous sections:

- leaving the match blank uses the previous search
- `~` represents the last replacement used

We're also making some assumptions that there was a previous substitution that set these values for us.

## Alternatives

You can do `:&&` which is the same as `&` but remembers the flags from the last substitute (`:help :&&`).
Note this one is an ex command.

There is also `g&` which applies the last substitution to the entire buffer (and remembers flags) (`:help g&`).

You might use `g&` in a situation where you've tested a substitution in an isolated context (or a different buffer)
and now want to quickly reapply the substitute to the current buffer.

# Alternative delimeters for substitute

We've always expressed `substitute` as `s/.../.../` but you don't actually have to use forward slashes to delimit
the different parts of the command.

You can use other characters like `#`, so long as you're consistent.

For example: `s#abc#ABC#g` replaces "abc" with "ABC" on the current line.

Then you could repeat it on another line by doing `s**~*g` which is like: `s//~/g`
(ie. use the last search and replace terms).

Why would you ever do this? Because you might be searching or replacing in a "forward slash heavy" context.
Rather than having to escape all those forward slashes,
just use a different delimeter that wouldn't otherwise appear in your pattern.

Both of these patterns replace "//" with "// ":

```
s/\/\//\/\/ /
s#//#// #
```

I think the second one is much easier to read!

