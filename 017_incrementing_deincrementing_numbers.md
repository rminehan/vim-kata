# Incrementing and de-incrementing numbers

Surprisinginly often I find myself needing to modify numbers inside a buffer I'm editing.

The `<c-a>` and `<c-x>` operator increment and de-increment the next number on the line respectively.

# Counts

These shortcuts can be used with a count, e.g. `10<c-a>` will increment the line by 10.

# Cursor position

Note that your cursor doesn't need to be on a number to increment it.

Vim will look for the next number on the line from where your cursor is.

It will also move the cursor to that line when you use the increment/de-increment operator.

# Floating points?

Text like "100.30" will be seen as two separate numbers "100" and "30" from the perspective of vim.

It won't see it as the floating point number 100.30

Vim is probably using "little word" boundaries to search for numbers.

Hence `<c-a>` with your cursor on the '3' would convert it to "100.31" and not "101.30" as it just sees 30 and is changing it to 31.

Likewise a number like "20,000" is actually "20" and "000" - so adding/subtracting will not have the effect you expect!

# Exercises

## Exercise 1

It's Enxhell's birthday - change his age in the sentence below:

- move your cursor to anywhere in "Happy 15"
- do `<c-a>`

```
Happy 16th birthday Enxhell!
```

## Exercise 2

Leadiq is giving a 10 dollar discount this month.

Change the price in the sentence below to reflect this:

- move your cursor anywhere before the price
- do `10<c-x>`

```
Leadiq professional edition now only costs $20000 USD per month.
```

## Exercise 3

The government is adding a tax on scala books so we need to up all the prices in this list by $10.

We could do this line by line but let's use this as a chance to use the normal command from [lesson 3](003_normal_commands.md).

- put your cursor on the first book listing
- do `:.,+4 normal 10<c-v><c-a>` which needs explaining:
    - `:` - enter command mode
    - `.,+4` - a range meaning the current line down to 4 lines below
    - `normal` - start a normal command
    - `10` - a count we're going to use with `<c-a>`
    - `<c-v>` - wait for a special key code (we can't just type literal characters "<c-a>"
    - `<c-a>` - the control a character

```
programming in scala - $40
practical scala - $60
scala for the impatient - $30
introduction to scala - $20
scala is 10 x better than python - $30
```

This will run `10<c-a>` on every line with the cursor at the start of each line (I'm pretty sure).

This worked for all our books except the last. Here the price stayed the same and the title changed to the one for my next book...

# Speed dating Tim Pope (every vimmer's dream)

Sometimes you have a date like `2020-01-31`.

It would be nice to be able to use `<c-a>` and `<c-x>` to increment the date by 1 day.

However vim sees it as 3 numbers. Putting your cursor on "31" and doing `<c-a>` will make the date `2020-01-32` :sad-parrot:

Tim Pope's ["speed dating" plugin](https://github.com/tpope/vim-speeddating) corrects this behavior by detecting dates
and changing the behavior of `<c-a>` and `<c-x>` to handle it as a date which in the example above would yield `2020-02-01` instead.

I don't work that much with dates in raw text so I haven't felt the need to install this plugin, but I thought it was worth mentioning.

# Octal numbers

Depending on how your vim is setup, a number like "07" might get interpreted as "octal" as it starts with a zero.

This means incrementing it would change it to "10" :upside-down-face:

See the `nrformats` option.

More info [here](https://vim.fandom.com/wiki/Increasing_or_decreasing_numbers).

Overall vim's support for octal probably comes back to it being used predominantly by C programmers early on,
who would work with octal literals in code.

The other usecase for octal support is that Enxhell can print his age in octal on his ID so that he can sneak into bars and clubs
(16 in decimal becomes 20 in octal).
