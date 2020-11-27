# vim-kata

Sets of 10 minute exercises to sharpen your vim skills.

The intention is that you clone this repo and work through the exercises with your team a few times a week (usually in the morning).

Over time your powers will grow, you'll obtain ultimate powers as your fingers are finally able to keep up with your massive brain.

# Doing the exercises

Each file represents an exercise. Simply open it in a vim-ish editor then follow the instructions to modify text within the file.

You might need to stash your changes before pulling updates.

# Editors

Generally the exercises will just revolve around out-of-the-box vim features and should be fine with:

- vi
- vim
- neovim
- spacemacs
- emacs with evil mode
- editors with decent vim plugins

In the cases where some special plugin or particular editor is needed the kata will make that clear at the top.

# Resetting to vanilla settings

If you're using a really sophisticated vim that has remapped keys, you can launch vim/nvim with `-u NONE` to return it to its stock settings:

```
nvim -u NONE README.md
```

# Markdown

Note that the kata are .md files, so whilst you don't have to have special support for rendering markdown, it might make the files a little easier to read.

Note that vim/nvim has a good markdown plugin [here](https://github.com/plasticboy/vim-markdown). It doesn't render the markdown, but provides some nice syntax highlighting (which is all you need here).

Most sophisticated editors like intellij and VSCode will have some kind of markdown plugin that highlights your markdown and can also render it in a split window.

# Normal mode

Normal mode in vim is called "normal" because it's the mode you're generally meant to be in (not insert mode).

The instructions in the kata generally implicitly assume you're in normal mode.

# Undoing

You might want to practice some exercises multiple times which means doing the exercise then undoing.

You can use `u` to undo changes.

Note that Tim Pope has a nice plugin vim repeat [here](https://github.com/tpope/vim-repeat) which (I think) also makes undo operations a little cleaner/faster. You don't need this for the exercises (regular undo is fine), but for some things it might make undo a bit more intuitive.

# Keyboard Input Conventions

The kata often specify keys to enter whilst using vim.

Generally they will be in `monospace code format`.

Usually the characters directly correspond to what you type, but there's some exceptions like:

- `<enter>` - enter
- `<escape>` - escape
- `<c-...>` - control + some key
- `<m-...>` - meta/alt/option + some key
- `<leader>` - whatever your leader key is

# Shoutout to "Practical Vim"

"Practical Vim" by Drew Neil is a great book for learning vim. I've read through it in bits and pieces 2-3 times.

A lot of the kata's, remarks and the general ordering of kata is heavily influenced by this book's approach.
