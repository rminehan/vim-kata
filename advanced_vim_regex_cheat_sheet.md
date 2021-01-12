# Advanced Vim Regex Cheat Sheet

Resources:

```
`:help pattern-searches`                       kata 43-51
https://neovim.io/doc/user/pattern.html        zero_width_matches.md
                                               too_lazy_for_laziness.md
```
# Common settings

```
ignorecase   case insensitive matching                           incsearch   shows matches interactively
smartcase    triggers case sensitivity on capital letter         inccommand  shows substitute effects interactively
hlsearch     highlights matches in the buffer
```

# Common flags

```
\v  very magic           \m  magic                            \c  case insensitive
\V  very no magic        \M  no magic                         \C  case sensitive

Applies to everything after it in the pattern                 Applies to the entire pattern regadless of position
```

# Limiting Scope with \%

```
\%V        match if inside most recent visual selection
\%#        match if at cursor position

:help /\%l                              :help /\%c                           :help /\%'
\%[LINE]l  match if on line             \%[COL]c  match if on column         \%'[MARK]  match if on mark's line
\%>[LINE]l match if after line          \%>[COL]c match if after column      \%>'[MARK] match if after mark's line
\%<[LINE]l match if before line         \%<[COL]c match if before column     \%<'[MARK] match if before mark's line
```

All zero width. See kata 43. No `\` needed in very magic mode.

# Look arounds with \@

```
[ATOM]\@[OPERATOR]

where OPERATOR is one of:

=   positive lookahead                   :help /\@=
!   negative lookahead  (first example)  :help /\@!
<=  positive lookbehind (second example) :help /\@<=
<!  negative lookbehind                  :help /\@<!
```

See kata 44. No `\` needed in very magic mode.

# Programmatically building patterns with vimscript

```
:execute  Executes a string as an ex command      line(...)    Function which returns a line number
.         Concatenates strings                    @[REGISTER]  Accesses the register
```

See kata 45.

# Language patterns 

```
Atom  Description  Controlled by            Each has an uppercase variant (\K,\I,\F) which excludes digits.
--------------------------------
\k    keyword      iskeyword
\i    ident        isident
\f    filename     isfilename
```

See kata 46, in particular for an explanation of how to control what characters get matched.

# Ampersand

```
[BRANCH 1]&[BRANCH 2]&[BRANCH 3] ... &[BRANCH n]            :help /\&

Requires all branches to match at a particular location.

Branches may consume different numbers of characters.

Branch n provides the final match to represent the group as a whole. 
```

See kata 47.

# Zoom operators

Quick way to select a subsection of your consumed text for a final match.

```
\zs   Start matching    :help /\zs
\ze   End matching      :help /\ze
```

See kata 47.

# Ranges

Quantifiers in the form `{...}` that attach to an atom.

```
{n}    Exactly n matches           Greedy by default
{n,}   At least n matches          Add '-' after '{' to make it lazy, e.g. {-3,}
{,m}   At most n matches           Lazy quantifiers can only be converted to greedy equivalents
{n,m}  n-m matches                 {}  is equivalent to *
{}     Any number of matches       {-} is equivalent to *? from perl
```

See kata 48 and too_lazy_for_laziness.md

# Multiline patterns

Standard atoms tend to not to match newlines, which limits matches to one line.

Many `\x` style atoms have an equivalent `\_x` which extends the atom to also match newline.

For example `\_.`, `\_s`, `\_d`. `:help /\_` shows a full list.

Beware: `\_.` with an unbounded greedy quantifier will consume your entire buffer before backtracking

See kata 49.

# Groups and back references

`\(...\)` creates a match group. Groups are labelled by number from left to right based on the opening bracket position.

`\1`, `\2` etc... are back references to the text matched by a group.

`\0` and `&` represent the entire match.

Vim supports at most 9 groups.

See kata 50.

# Case modification

Flags exist to modify the case of text used for replacement. `:help sub-replace-special`

```
\u  uppercase next character       \l  lowercase next character
\U  uppercase until \E             \L  lowercase until \E
```

See kata 50.

# Shortcuts to reduce typing

- omitting a pattern uses what's in register `/`
- `~` is the last substituted string (doesn't work well with `inccommand`)
- `<c-f>` opens the history in a buffer from search and ex command mode (see also fzf's `History:` and `History/`)
- `&` from normal mode redoes the last substitute with no flags against the current line
- `:&&` redoes the last substitute against the current line (and remembers flags)
- `g&` redoes the last substitute against the whole buffer (and remembers flags)

See kata 51.
