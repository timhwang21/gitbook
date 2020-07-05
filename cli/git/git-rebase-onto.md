# git rebase-onto

## Problem

You have a branch `feature-a`. You finish it and submit it for review, and then branch off of `feature-a` to start working on `feature-b` which depends on your latest changes.

Your code is reviewed and some changes are requested. You update `feature-a`, push, and merge. You want to keep `feature-b` up to date, so you check it out and run `git rebase origin/master`. However, your latest revisions lead to a self merge conflict when trying to rebase `feature-b` off of master.

```text
# Before code review
# origin/master
master
# feature-a
master -> feature-a
# feature-b
master -> feature-a -> feature-b

# After code review
# origin/master (using master as parent for consistency, though HEAD is now feature-a')
master -> feature-a' # revised feature-a is now HEAD
# feature-b (feature-a conflicts with feature-a')
master -> feature-a -> feature-b
```

## Solution

Instead of `git rebase origin/master`, use `git rebase --onto origin/master feature-a`.

Specifically, this means "take all commits that are on top of `feature-a`, and place them on top of `origin/master`'s `HEAD`." In the common scenario where `feature-a` is the only difference between `origin/master` and `feature-b`, it can be thought of as "drop all changes between `origin/master`'s `HEAD` and `feature-a` inclusive."

