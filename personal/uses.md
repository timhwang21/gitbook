# Uses

## Gallery

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim.png)

Standard Vim editor window (window size slightly reduced for better screenshotting).

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim-coc.png)

Tapping into [tsserver](https://github.com/Microsoft/TypeScript/wiki/Standalone-Server-%28tsserver%29) with coc.nvim to enable IDE-like functionality. The "floating window" functionality is provided by Neovim.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim-fzf-file-search.png)

Using FZF as a fuzzy file opener. Much faster than Ctrl-P, because file indexing happens asynchronously.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim-fzf-git-log-p.png)

Using FZF to run an analogue of `git log -p` to quickly search a file's history.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim-diff.png)

Diffing files (and resolving merge conflicts) within Vim.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/fira-code-ligatures.png)

Demonstration of Fira Code ligatures, as seen in Haskell. `<$>` in particular is beautifully rendered.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/tig.png)

Staging files with Tig.

## Editor

Currently, Vim (specifically [Neovim](https://neovim.io/)) is my editor of choice. This was largely by necessity, as a previous job required running a Dockerized blockchain during development, which made all but the most lightweight editors unusable. Plus, nothing gives quite the same satisfaction as procrastinating by tinkering with Vim settings.

My setup can be found [here](https://github.com/timhwang21/dotfiles/blob/master/settings/.vimrc).

### Plugins

Several plugins I find indispensible:

- [janko/vim-test](https://github.com/janko/vim-test) - Run specs asynchronously without leaving the editor.
- [junegunn/fzf.vim](https://github.com/junegunn/fzf.vim) - Incredibly flexible fuzzy file finder. I use it for opening files, searching open buffers, searching files' git history, navigating between unstaged changes... the list goes on.
- [neoclide/coc.nvim](https://github.com/neoclide/coc.nvim) - In my opinion, this is the most important plugin I has installed. Enables language server support for Vim, allowing for powerful, IDE like features like definition navigation, bulk renaming, etc.
- [scrooloose/nerdtree](https://github.com/scrooloose/nerdtree) - I use this with [vim-devicons](https://github.com/ryanoasis/vim-devicons) to show filetype-based symbols by the filenames. It's aesthetically pleasing and has ergonomic shortcuts.
- [sheerun/vim-polyglot](https://github.com/sheerun/vim-polyglot) - A fairly high quality aggregator of language configurations. I've found that it "Just Works ™" when opening files of any type, no matter how arcane.
- Many, many Tim Pope plugins.

### Theme

I started coding with [Sublime Text](https://www.sublimetext.com/3dev), and [Monokai](https://neovim.io/) has never not been the most natural appearance of code for me.

### Font

I use the [NerdFont](https://github.com/ryanoasis/nerd-fonts) version of [Fira Code](https://github.com/ryanoasis/nerd-fonts/tree/master/patched-fonts/FiraCode), which is a font with ligatures. Notably, it makes ML-derived languages look incredibly beautiful.

## Terminal

I currently use Bash (my dotfiles can be found [here](https://github.com/timhwang21/dotfiles/blob/master/settings/.bashrc)). I've been telling myself to switch to Zsh for over 3 years, but my current setup is so ingrained into my muscle memory that I've been putting it off indefinitely.

I use [iTerm2](https://iterm2.com/) as my terminal, though I'm currently evaluating [kitty](https://github.com/kovidgoyal/kitty) and [alacritty](https://github.com/alacritty/alacritty). Both are GPU-accelerated, which you wouldn't think is needed for a terminal emulator, but the difference is quite stunning. I will probably never go back.

I've also listed some specific tools I particularly enjoy [here](cli/tools.md).

