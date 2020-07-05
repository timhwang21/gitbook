# Tools

In my opinion, optimizing and building proficiency in the tools you use is one of the highest rate of return activities you can do as an engineer. Good tools are productivity multipliers, and give their benefit across different contexts.

Below are several tools that stand out to me as being both incredibly useful and not necessarily well-known \(or have benefits that aren't easily conveyed\).

For a full list of the tools I use, take a look at my [dotfiles](https://github.com/timhwang21/dotfiles/blob/master/bootstrap-download.sh).

## [asciinema](https://github.com/asciinema/asciinema) - terminal recorder

asciinema is a terminal recorder that records your buffer output as a text file rather than as a video file. The most immediate advantage is that viewers can copy and paste code directly from the recorded cast. It's also a great teaching tool for demonstrating efficient workflows.

Additionally, it's also incredibly resource light compared to screen sharing or screen recording, which are both nightmares if I'm trying to concurrently run something expensive. A quick terminal recording is way faster and more lightweight.

## [coc.nvim](https://github.com/neoclide/coc.nvim) with [Neovim](https://github.com/neovim/neovim), [Ale](https://github.com/dense-analysis/ale) and a [LSP](https://en.wikipedia.org/wiki/Language_Server_Protocol)

* LSP: a standardized protocol for **language servers**, editor-agnostic providers for language-specific functionality
* coc.nvim: LSP interface for Vim
* Ale: LSP-powered linter for Vim
* Neovim: Vim fork needed to leverage certain powerful coc functionalities

I switched to Vim as my primary editor about a year ago out of necessity. Our Dockerized backend setup was so heavy that when running everything locally with our app open in Chrome, pretty much anything with a GUI would grind to a halt. With VSCode being utterly unusable, I started to research how to replicate its Typescript developer experience in Vim. It turns out that the combination of Coc, Neovim, and Microsoft's own Typescript LSP were enough to finally make Vim feel like a full-fledged IDE.

LSPs are the servers that power certain language features for VSCode such as showing inline type declarations, jumping to definitions, and advanced code completion. Coc brings this power to Vim, and Neovim utilizes this newfound power to its fullest. Finally, Ale adds linting that doesn't cause the editor to freeze by leveraging Vim's new\(ish\) async functionalities. My editor no longer competes with everything else for system resources, and I enjoy all of VSCode's advanced features straight from Vim.

As a usage example, I have `<F7>` and `<F8>` bound to `coc-diagnostic-prev` and `coc-diagnostic-next`, and `K` bound to a function to show documentation in a Neovim popup, replicating VSCode behavior. `gd` jumps to any definition, and `<Leader>rn` renames any symbol globally throughout a codebase -- no more regex-based find-replacing!

The one pain point is that it can be difficult to integrate with existing Vim setups, as many plugins are quite opinionated. Coc also has its own extension ecosystem, which can be intimidating to wade through. But the final result is oh so worth it.

## [fasd](https://github.com/clvv/fasd) - full featured quickjump utility

Autojump and rupa/z are great tools, so great that almost everyone who picks them up are satisfied forever. fasd is a small incremental enhancement on top of these tools, with slightly more sophisticated commands that offer more flexibility. Instead of being limited to `cd`ing around, fasd can return fuzzy matches for files and directories. This is super convenient and allows for shortcuts like the following:

```bash
# the subshell will expand to the best matching file
vim `f some_file`

# will expand to the best matching file and move that to the best matching directory
mv `f some_file` `d some_directory`
```

There is also an interactive selection CLI for multiple matches, but in all honesty I have not used it a single time ever.

## [tig](https://github.com/jonas/tig/) - CLI interface for git

By far the best interface I've used for Git among CLIs and GUIs. Lightning quick, and intuitive to use.

I generally dislike GUI git interfaces as I find them both overwhelming and slow. It seems the features GUI users value the most is easy merge conflict resolution and easy interactive staging. Tig doesn't handle the former at all \(I use `vim-diff` for this\), and is flat-out amazing at the latter. With its large array of shortcuts, it makes carefully staging clean commits a breeze. Additionally, I use this less often, but the cherry-picking interface is extremely user friendly as well -- no more dealing with commit SHAs. Finally, the log view is gorgeous.

## [vim-test](https://github.com/janko/vim-test)

Run tests directly from the editor. You can run the test under your cursor, run all tests in a file, and rerun the last tested file from anywhere, among other strategies. Has modes that leverage Neovim features which results in honestly a very slick workflow -- make some changes, call `:TestVisit`, see the results appear in a popup, and dismiss the popup and get right back to coding.

