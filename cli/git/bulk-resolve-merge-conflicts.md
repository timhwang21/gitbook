# Bulk resolve merge conflict

## Problem

You have a branch that conflicts with `master` in small but numerous occasions. Manually fixing each would be repetitive. Even running `git checkout --theirs -- [filename]` on each would be repetitive.

## Solution

We need several things here:

1. A way to get a list of all conflicted files
2. A way to interactively filter out the files we don't want (it's unlikely you know exactly which files have trivial diffs beforehand)
3. `xargs` to pass these into `git rebase`

A naive approach would be to use the output of `git diff`. We use the `--no-pager` option to `cat` the output for use in pipes.

```bash
git --no-pager diff --name-only | fzf | xargs git checkout --theirs --
```

However, this does not distinguish between regular diffs and conflicts.

[Looking at the docs](https://git-scm.com/docs/git-diff), it seems like adding the `--check` flag to `git diff` does what we want. However, `--check` also flags whitespace markers for some reason, which can add noise, and it adds text after the filename, which must be split out (and I'm bad at `awk`).

The `--diff-filter` option is more promising. With `--diff-filter=U`, we only list unmerged files, and when rebasing, only conflicts are shown.

So, our final solution is:

```bash
git --no-pager diff --name-only --diff-filter=U | fzf | xargs git checkout --theirs --
```

### Facebook Pathpicker

There's an interesting file picker tool from Facebook called [PathPicker](https://github.com/facebook/PathPicker). Rather than operating on lines `grep`-like tools, `fpp` parses input and extracts path-like entities. This doesn't filter out non-conflicted diffs, but at least the filenames will be visually organized and sectioned like they are from `git status`. Thus, our command becomes:

```bash
git status | fpp
```

This approach relies more heavily on a UI, but is simpler.

### The nuclear approach

When you are sure that all conflicts are trivial, there is a blunter, simpler solution:

```bash
git rebase --abort
git rebase -Xtheirs origin/master
```

## Bonus tip

While figuring this out, I also stumbled across `git checkout --conflict`, which resets merge conflict markers in case you made a mistake and want to start the resolution from scratch.
