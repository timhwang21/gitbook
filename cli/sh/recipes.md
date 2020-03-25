# Recipes

Useful workflows that rely on a combination of tools.

## Diff certain files that have changed on a branch

```sh
vim -c 'tabdo Gdiff origin/master' -p $(git diff --name-only origin/master | fzf)
```

{% hint style="info" %}
[I've founf this useful enough to add as a git alias](https://github.com/timhwang21/dotfiles/blob/master/settings/.gitconfig#L34). The above implementation diffs against the branch point insttead of `origin/master` which results in a cleaner diff.
{% endhint %}

### Explanation

When reviewing a feature branch, sometimes you want to know how one specific feature was impacted by the branch. For example, you want to see how every file in a certain directory or with a certain name was changed.

We open Vim with the command "for each tab (`tabdo`), diff the file against its version on master (`Gdiff origin/master`), and then we pass it the list of all files changed in the current branch (`git diff --name-only origin/master`), filtered using `fzf`."

What happens is that `fzf`'s prompt opens up, letting you select a list of files using `Tab`. Upon completing your selection, all files will be diffed in splits in separate tabs, which you can navigate between using `gt`.

### Tools used

- `vim-fugitive` (specifically, `:Gdiff`)
- `fzf`
