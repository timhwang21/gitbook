# Patch from diff

Normally when using `git apply`, you first generate a patch file with `git format-patch`. However, sometimes you want to send a quick diff to someone with `git diff FILE | pbcopy`.

Trying to pipe this into `git apply` will give a "corrupt patch" error, because `git diff` doesn't give the trailing newline that `git format-patch` does. We can fix this by wrapping in `echo`:

```sh
git diff FILE | pbcopy
# send to coworker; coworker runs following
echo "$(pbpaste)" | git apply
```
