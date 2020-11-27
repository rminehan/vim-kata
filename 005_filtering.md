# Filtering

So far we've encountered vim operations like `d` (delete), `c` (change) and `y` (yank/copy).

These can be combined with text objects for powerful editing.

Today we're adding another powerful operator to the list: `!` (filter).

Filtering takes your text object, feeds it through an external program, then replaces the current text object with the output.
Easiest to show through example.

The syntax is `![TEXT OBJECT][COMMAND]`.

The first part `![TEXT OBJECT]` should be familiar to us - then you're prompted to type in the specific program to run.

# Exercises

Note: The exercises assume you have standard tools like `sort` and `grep` installed.
This should be the case for macos/linux.
If you're on windows and having issues see [these instructions](help_for_windows_users.md)

We also use `jq` which isn't standard but it's good for you to install it as it's so useful.

## Exercise 1 - sorting

Use filtering to sort the paragraph of names below.

- put your cursor anywhere in the paragraph below
- do `!ipsort<enter>` which breaks down to:
  - `!` - the operator
  - `ip` - inside paragraph
  - `sort` - the external program to filter the text through

```

boban
joban
clement
thilo
paul
gagan
vish
rohan
zij
lulu
jonathan
james

```

(Note that vim also has a built-in `:sort` command)

## Exercise 2 - grep-ing

HR has mandated a limit of 3 people in the engineering team who are allowed "j" in their name.

Figure out who they are so we can pick who to fire:

- put your cursor anywhere in the paragraph below
- do `!ipgrep j<enter>` which breaks down to:
  - `!` - the operator
  - `ip` - inside paragraph
  - `grep j` - uses the grep tool to match any text containing "j"

```

boban
joban
clement
thilo
paul
gagan
vish
rohan
zij
lulu
jonathan
james
joey-jo-jo-junior-shabadoo
mei

```

Joey Jo Jo wait!

## Exercise 3 - jq

We got back a one-line json response from an api.

Use filtering and `jq` to prettify it into standard multi-line json.

- put your cursor anywhere on the line
- do `!ipjq .` which breaks down to:
  - `!` - the operator
  - `ip` - inside paragraph
  - `jq .` - prettify json using `jq`

```json

{"fire": ["joban", "joey-jo-jo-junior-shabadoo"], "dontFire": ["zij", "james", "jonathan"]}

```

## Exercise 4 - figlet

For those who wonder how Rohan generates:

```
       _             _     _            _   
  __ _(_) __ _ _ __ | |_  | |_ _____  _| |_ 
 / _` | |/ _` | '_ \| __| | __/ _ \ \/ / __|
| (_| | | (_| | | | | |_  | ||  __/>  <| |_ 
 \__, |_|\__,_|_| |_|\__|  \__\___/_/\_\\__|
 |___/                                      
```

in his slides,

I filter it through the `figlet` program.

# Summary: Filtering and cli tools

Filtering is extremely powerful because it lets you leverage tools you already _should_ know from the command line (grep, jq, sort, head, ...).

This means we don't need to create plugins that duplicate tools that we know and love.

It also means that learning how to use CLI tools will make you faster in vim. More and more you can stay in the warm comfy terminal.

This gives vim an edge over most other editors (my guess is emacs provides something like this).
