# Uses

## Gallery

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim.png)

Standard Vim editor window.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim-coc.png)

Tapping into [tsserver](https://github.com/Microsoft/TypeScript/wiki/Standalone-Server-%28tsserver%29) with [coc.nvim](https://github.com/neoclide/coc.nvim) to enable IDE-like functionality. The "floating window" functionality is provided by Neovim.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim-fzf-file-search.png)

Using [fzf](https://github.com/junegunn/fzf) (and [fzf.vim](https://github.com/junegunn/fzf.vim)) as a fuzzy file opener. Much faster than Ctrl-P, because file indexing happens asynchronously.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim-fzf-git-log-p.png)

Using fzf to run an analogue of `git log -p` to quickly search a file's history.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/vim-diff.png)

Diffing files \(and resolving merge conflicts\) within Vim.

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/tig.png)

Staging files with [Tig](https://github.com/jonas/tig).

![](https://raw.githubusercontent.com/timhwang21/gitbook/master/assets/images/wtfutil.png)

[WTF](https://wtfutil.com/) terminal dashboard.

## Editor

[Neovim](https://neovim.io).

My setup can be found [here](https://github.com/timhwang21/dotfiles/blob/master/settings/.vimrc).

### Plugins

Several plugins I find indispensible:

* [janko/vim-test](https://github.com/janko/vim-test) - Run specs asynchronously without leaving the editor.
* [junegunn/fzf.vim](https://github.com/junegunn/fzf.vim) - Incredibly flexible fuzzy file finder. I use it for opening files, searching open buffers, searching files' git history, navigating between unstaged changes... the list goes on.
* [neoclide/coc.nvim](https://github.com/neoclide/coc.nvim) - In my opinion, this is the most important plugin I has installed. Enables language server support for Vim, allowing for powerful, IDE like features like definition navigation, bulk renaming, etc.
* [scrooloose/nerdtree](https://github.com/scrooloose/nerdtree) - I use this with [vim-devicons](https://github.com/ryanoasis/vim-devicons) to show filetype-based symbols by the filenames. It's aesthetically pleasing and has ergonomic shortcuts.
* [sheerun/vim-polyglot](https://github.com/sheerun/vim-polyglot) - A fairly high quality aggregator of language configurations. I've found that it "Just Works ™" when opening files of any type, no matter how arcane.
* Many, many Tim Pope plugins.

### Theme

I started coding with [Sublime Text](https://www.sublimetext.com/3dev), and [Monokai](https://neovim.io/) has never not been the most natural appearance of code for me.

### Font

I use the [NerdFont](https://github.com/ryanoasis/nerd-fonts) version of [JetBrains Mono](https://github.com/ryanoasis/nerd-fonts/tree/master/patched-fonts/JetBrainsMono), which is a font with ligatures. It's not the most aesthetically pleasing font, but it is incredibly readable.

## Terminal

I currently use Bash \(my dotfiles can be found [here](https://github.com/timhwang21/dotfiles/blob/master/settings/.bashrc)\). I've been telling myself to switch to Zsh for over 3 years, but my current setup is so ingrained into my muscle memory that I've been putting it off indefinitely.

I use [kitty](https://sw.kovidgoyal.net/kitty/). It GPU-accelerated, which you wouldn't think is needed for a terminal emulator, but the difference is quite stunning. I will probably never go back. Also, the configuration is incredibly friendly.

I've also listed some specific tools I particularly enjoy [here](https://github.com/timhwang21/gitbook/tree/c3d91ebb46c5494ca20a12559ef13f64d30fd5cb/personal/cli/tools.md).

